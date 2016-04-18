---
layout: post
title: Dynamically generate class and set its fields value
comments: true
author: "Yin Haomin"
tags: 
  - Bean
  - Dynamic class
  - javassist
---

我们的需求场景是这样的，对于一个数据中间件，每次上线都要重新写代码，而作为一个透传数据的中间件，完全没有必要写新的代码，可以改造成仅仅需要配置就可以了。那么如何将订阅配置化，如何动态的生成class，并根据传入的类型设置其property值。

大体的逻辑如下:

![gras](/images/postsImages/Change-dumper改造设计.png)

以下是我最终实现的可以运行的方式。

#### 如何动态的生成Class

```
import java.io.Serializable;
import java.util.Map;
import java.util.Map.Entry;

import javassist.CannotCompileException;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtField;
import javassist.CtMethod;
import javassist.NotFoundException;

/**
 * 生成Class的类
 * 
 * @author Yin Haomin
 * @date 2016/04/18
 *
 */
@SuppressWarnings("rawtypes")
public class ClassGenerator {

    /**
     * 生成Calss
     * 
     * @param className
     * @param properties
     * @return
     * @throws NotFoundException
     * @throws CannotCompileException
     */
    public static Class generate(String className, Map<String, Class<?>> properties) throws NotFoundException,
            CannotCompileException {

        ClassPool pool = ClassPool.getDefault();
        CtClass cc = pool.makeClass(className);

        // add this to define a super class to extend
        // cc.setSuperclass(resolveCtClass(MySuperClass.class));

        // add this to define an interface to implement
        cc.addInterface(resolveCtClass(Serializable.class));

        for (Entry<String, Class<?>> entry : properties.entrySet()) {

            cc.addField(new CtField(resolveCtClass(entry.getValue()), entry.getKey(), cc));

            // add getter
            cc.addMethod(generateGetter(cc, entry.getKey(), entry.getValue()));

            // add setter
            cc.addMethod(generateSetter(cc, entry.getKey(), entry.getValue()));
        }

        return cc.toClass();
    }

    /**
     * 生成get方法
     * 
     * @param declaringClass
     * @param fieldName
     * @param fieldClass
     * @return
     * @throws CannotCompileException
     */
    private static CtMethod generateGetter(CtClass declaringClass, String fieldName, Class fieldClass)
            throws CannotCompileException {

        String getterName = "get" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);

        StringBuffer sb = new StringBuffer();
        sb.append("public ").append(fieldClass.getName()).append(" ").append(getterName).append("(){").append(
                "return this.").append(fieldName).append(";").append("}");
        return CtMethod.make(sb.toString(), declaringClass);
    }

    /**
     * 生成set方法
     * 
     * @param declaringClass
     * @param fieldName
     * @param fieldClass
     * @return
     * @throws CannotCompileException
     */
    private static CtMethod generateSetter(CtClass declaringClass, String fieldName, Class fieldClass)
            throws CannotCompileException {

        String setterName = "set" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);

        StringBuffer sb = new StringBuffer();
        sb.append("public void ").append(setterName).append("(").append(fieldClass.getName()).append(" ").append(
                fieldName).append(")").append("{").append("this.").append(fieldName).append("=").append(fieldName)
                .append(";").append("}");
        return CtMethod.make(sb.toString(), declaringClass);
    }

    private static CtClass resolveCtClass(Class clazz) throws NotFoundException {
        ClassPool pool = ClassPool.getDefault();
        return pool.get(clazz.getName());
    }
}

```

#### 顺序的读取配置文件中的配置并测试一下生成的class

```
public class ClassUtil {

    /**
     * 根据传入的folderPat，读取出来所有的配置文件，按照配置文件的配置生成类，并将需要的信息写入到相应的map中
     * 
     * @param folderPath
     *            存放类的配置文件的路径
     * @param objectPropertyMap
     *            存放配置文件的property的map
     * @param objectMap
     *            存放object的map
     * @throws Exception
     */
    public static void generateDynamicClass(String folderPath, Map<Integer, Map<String, Integer>> objectPropertyMap,
            Map<Integer, Class> classMap) throws Exception {

        URL path = ClassUtil.class.getClassLoader().getResource(folderPath);
        File filePath = new File(path.getPath());
        File[] roleFiles = filePath.listFiles();

        for (int k = 0; k < roleFiles.length; k++) {
            try {
                // 文件名称即为CDP文件的type信息，这里的sequence等于CDP文件的type信息
                String fileName = roleFiles[k].getName();
                String sequenceStr = fileName.split("\\.")[0];
                int sequence = Integer.parseInt(sequenceStr);
                HashMap returnMap = new HashMap();
                HashMap typeMap = new HashMap();
                // 读取配置文件
                Properties prop = new OrderedProperties();
                InputStream inputStream = ClassUtil.class.getClassLoader().getResourceAsStream(
                        "roles/" + roleFiles[k].getName());
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream, "utf-8"));
                prop.load(bufferedReader);
                inputStream.close();
                Set<String> keylist = prop.stringPropertyNames();
                BeanInfo beanInfo = Introspector.getBeanInfo(Role.class);
                PropertyDescriptor[] propertyDescriptors = beanInfo.getPropertyDescriptors();

                for (int i = 0; i < propertyDescriptors.length; i++) {
                    PropertyDescriptor descriptor = propertyDescriptors[i];
                    String propertyName = descriptor.getName();
                    if (!propertyName.equals("class")) {
                        Method readMethod = descriptor.getReadMethod();
                        Object result = readMethod.invoke(new Role(), new Object[0]);
                        if (result != null) {
                            returnMap.put(propertyName, result);
                        } else {
                            returnMap.put(propertyName, "");
                        }
                        typeMap.put(propertyName, descriptor.getPropertyType());
                    }
                }
                // 加载配置文件中的属性
                Iterator<String> iterator = keylist.iterator();
                Map<String, Integer> propertiesMap = Maps.newHashMap();
                int propertySequence = 1;
                while (iterator.hasNext()) {
                    String key = iterator.next();
                    propertiesMap.put(key, propertySequence);
                    propertySequence++;
                    returnMap.put(key, prop.getProperty(key));
                    typeMap.put(key, Class.forName(returnMap.get(key).toString()));
                }
                // // map转换成实体对象
                // DynamicBean bean = new DynamicBean(typeMap);

                Class<?> clazz = ClassGenerator.generate("net.javaforge.blog.javassist.Pojo$Generated" + sequence,
                        typeMap);

                Object obj = clazz.newInstance();

                System.out.println("Clazz: " + clazz);
                System.out.println("Object: " + obj);
                System.out.println("Serializable? " + (obj instanceof Serializable));

                for (final Method method : clazz.getDeclaredMethods()) {
                    System.out.println(method);
                }

                // // 赋值
                // Set keys = typeMap.keySet();
                // for (Iterator it = keys.iterator(); it.hasNext();) {
                // String key = (String) it.next();
                // // 使用默认值0作为通用数据传入到bean中.
                // bean.setValue(key, 0, returnMap.get(key).toString());
                // }
                // Object object = bean.getObject();
                // 将相应的数值存放到两个map中，之后会用两个map.
                objectPropertyMap.put(sequence, propertiesMap);
                classMap.put(sequence, clazz);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```

#### 使用动态生成的Class

```
        Class clazz = RolesGenerator.classMap.get(ProcessConstant.ITEM_TYPE);
        Map<String, Integer> propertyNameMap = RolesGenerator.objectPropertyMap.get(ProcessConstant.ITEM_TYPE);

        Object object = clazz.newInstance();
        Field[] nameFields = object.getClass().getDeclaredFields();
        for (Field field : nameFields) {
            String name = field.getName();
            int columnSequence = propertyNameMap.get(name);
            StringBuilder sb = new StringBuilder(name);
            sb.setCharAt(0, Character.toUpperCase(sb.charAt(0)));
            Object value = ClassUtil.convert(field.getType(), fields[columnSequence + 2]);

            // set property
            clazz.getMethod("set" + sb.toString(), field.getType()).invoke(object, value);
        }

        Long id = (Long) clazz.getMethod("getId").invoke(object);
```

#### 最后，一种用CGLIB生成动态Class的方式，似乎是不能添加method，不好用

```
public class DynamicBean {

    // 动态生成的类
    public Object object = null;

    // 存放属性名称以及属性的类型
    public BeanMap beanMap = null;

    public DynamicBean() {
        super();
    }

    @SuppressWarnings("rawtypes")
    public DynamicBean(Map propertyMap) {
        this.object = generateBean(propertyMap);
        this.beanMap = BeanMap.create(this.object);
    }

    /**
     * 给bean属性赋值
     * 
     * @param property
     *            属性名
     * @param value
     *            值
     */
    public void setValue(Object property, Object value, String clazzType) {
        Object parsedValue = parseValue(value, clazzType);
        beanMap.put(property, parsedValue);
    }

    /**
     * 根究传入的基本数据类型解析数据
     * 
     * @param value
     * @param clazzType
     * @return
     */
    private Object parseValue(Object value, String clazzType) {
        Object result = new Object();
        if (clazzType.equals("java.lang.String")) {
            result = String.valueOf(value);
        } else if (clazzType.equals("java.lang.Long")) {
            result = Long.parseLong(value.toString());
        } else if (clazzType.equals("java.lang.Integer")) {
            result = Integer.parseInt(value.toString());
        } else {
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.append("The input type: ").append(clazzType).append(
                    " cannot be parsed. And the value is: " + value);
            log.error(stringBuilder.toString());
        }
        return result;
    }

    /**
     * 通过属性名得到属性值
     * 
     * @param property
     *            属性名
     * @return 值
     */
    public Object getValue(String property) {
        return beanMap.get(property);
    }

    /**
     * 得到该实体bean对象
     * 
     * @return
     */
    public Object getObject() {
        return this.object;
    }

    /**
     * 使用CGLIB的Generator生成一个bean.
     * 
     * @param propertyMap
     * @return
     */
    @SuppressWarnings("rawtypes")
    private Object generateBean(Map propertyMap) {
        BeanGenerator generator = new BeanGenerator();
        Set keySet = propertyMap.keySet();
        for (Iterator iterator = keySet.iterator(); iterator.hasNext();) {
            String key = (String) iterator.next();
            generator.addProperty(key, (Class) propertyMap.get(key));
        }
        return generator.create();
    }

}
```
