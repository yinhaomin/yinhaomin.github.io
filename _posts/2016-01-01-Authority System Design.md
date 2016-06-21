---
layout: post
title: Authority System Design
comments: true
author: "Yin Haomin"
keywords: 权限,权限系统,系统设计,Authority System Design
---

完成了一个权限管理系统,能够进行RBAC。
需对后台所有功能模块及功能进行职责权限划分，通过权限管理可配置不同权限给到相应人员。

建立了5张表: role, auth, role_auth, admin_user, admin_user_role.

role表

```
CREATE TABLE `role` (
	`id` INT(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '角色id',
	`name` VARCHAR(128) NOT NULL DEFAULT '' COMMENT '角色名称',
	`addtime` DATETIME NOT NULL DEFAULT '2015-09-01 00:00:00' COMMENT '创建时间',
	`updatetime` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
	PRIMARY KEY (`id`),
	UNIQUE INDEX `uk_name` (`name`)
)
COMMENT='角色表'
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=60
;
```

role_auth表

```
CREATE TABLE `role_auth` (
	`id` INT(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
	`role_id` INT(32) NOT NULL DEFAULT '0' COMMENT '角色id',
	`auth_id` INT(32) NOT NULL DEFAULT '0' COMMENT '权限id',
	PRIMARY KEY (`id`),
	INDEX `idx_role_id` (`role_id`),
	INDEX `idx_auth_id` (`auth_id`)
)
COMMENT='角色权限分配表'
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=670
;
```

auth表

```
CREATE TABLE `auth` (
	`id` INT(32) UNSIGNED NOT NULL COMMENT '权限id',
	`name` VARCHAR(128) NOT NULL DEFAULT '' COMMENT '权限展示名称',
	`parent_id` INT(32) NOT NULL DEFAULT '0' COMMENT '父权限id',
	`comment` VARCHAR(256) NOT NULL DEFAULT '' COMMENT '权限说明',
	PRIMARY KEY (`id`)
)
COMMENT='权限表'
COLLATE='utf8_general_ci'
ENGINE=InnoDB
;

```

admin_user表

```
CREATE TABLE `admin_user` (
	`uc_id` INT(32) UNSIGNED NOT NULL COMMENT '用户的ucid',
	`name` VARCHAR(256) NOT NULL DEFAULT '' COMMENT '用户名',
	`email` VARCHAR(256) NOT NULL DEFAULT '' COMMENT '用户email',
	`addtime` DATETIME NOT NULL DEFAULT '2015-12-01 00:00:00' COMMENT '用户添加时间',
	`updatetime` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
	PRIMARY KEY (`uc_id`)
)
COMMENT='UC用户信息表'
COLLATE='utf8_general_ci'
ENGINE=InnoDB
;

```

admin_user_role

```
CREATE TABLE `admin_user_role` (
	`id` INT(32) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
	`role_id` INT(32) NOT NULL COMMENT '角色id',
	`uc_id` INT(32) NOT NULL COMMENT '权限id',
	PRIMARY KEY (`id`),
	UNIQUE INDEX `uk_role_uc_id` (`role_id`, `uc_id`)
)
COMMENT='uc用户角色关系表'
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=130
;
```

各个表的初始化内容

```
INSERT INTO `admin_user` (`uc_id`, `name`, `email`, `addtime`, `updatetime`) VALUES
	(1422795, 'dltest', 'dltest@baidu.com', '2016-01-21 14:28:28', '2016-01-21 14:28:28'),
	(17908814, 'fuluchii', 'zhaofeifei@baidu.com', '2015-11-23 00:00:00', '2015-11-23 14:26:21'),
	(17908816, 'fuluchii2', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 09:49:39'),
	(17908899, 'fuluchii4', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 16:30:39'),
	(17908900, 'fuluchii7', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 16:37:32'),
	(17908901, 'fuluchii8', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 16:37:45'),
	(17908902, 'fuluchii9', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 16:38:01'),
	(17908903, 'fuluchii10', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 16:38:14'),
	(17908904, 'fuluchii11', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 16:38:40'),
	(17908905, 'fuluchii12', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 16:38:53'),
	(17908906, 'fuluchii13', 'zhaofeifei@baidu.com', '2015-11-18 00:00:00', '2015-11-18 16:39:17'),
	(18524727, 'lihaihua02', 'lihaihua02@baidu.com', '2015-12-16 11:30:10', '2015-12-16 11:30:10'),
	(18524838, 'hui-haoxiao', 'haoxiao01@baidu.com', '2015-12-16 11:30:10', '2015-12-16 11:30:10'),
	(18526547, 'liqian18', 'liqian18@baidu.com', '2015-12-16 11:30:10', '2015-12-16 11:30:10'),
	(18527757, '超级百度惠', 'haoxiao01@baidu.com', '2015-12-16 11:30:10', '2015-12-16 11:30:10');

	
INSERT INTO `admin_user_role` (`id`, `role_id`, `uc_id`) VALUES
	(115, 1, 1422795),
	(128, 1, 17908816),
	(98, 1, 18524727),
	(99, 1, 18524838),
	(100, 1, 18526547),
	(101, 1, 18527757),
	(116, 2, 1422795),
	(62, 2, 17908905),
	(63, 2, 17908906),
	(117, 30, 1422795),
	(61, 30, 17908904),
	(118, 42, 1422795),
	(102, 42, 17908814),
	(119, 44, 1422795),
	(103, 44, 17908814),
	(52, 44, 17908899),
	(120, 45, 1422795),
	(53, 45, 17908899),
	(60, 45, 17908903),
	(121, 46, 1422795),
	(122, 49, 1422795),
	(71, 49, 17908902),
	(123, 50, 1422795),
	(72, 50, 17908901),
	(124, 51, 1422795),
	(127, 57, 17908900);

INSERT INTO `auth` (`id`, `name`, `parent_id`, `comment`) VALUES
	(0, 'root', 0, '隐形的根id'),
	(1, '爆料管理', 0, '爆料列表tab是否可见'),
	(2, '已发布信息管理', 0, '已发布信息tab是否可见'),
	(3, '晒单经验管理', 0, '晒单经验管理tab是否可见'),
	(4, '竞品优惠管理', 0, '竞品优惠管理tab'),
	(5, '信息管理', 0, '信息管理tab'),
	(6, '积分管理', 0, '积分管理tab是否可见'),
	(7, '用户管理', 0, '用户管理tab是否可见'),
	(8, '评论管理', 0, '评论管理tab是否可见'),
	(9, '勋章管理', 0, '勋章管理tab是否可见'),
	(10, '公告管理', 0, '公告管理tab是否可见'),
	(11, '标签管理', 0, '标签管理tab是否可见'),
	(12, '抽奖管理', 0, '抽奖管理tab是否可见'),
	(13, '百度千寻', 0, '百度千寻是否可见'),
	(14, 'app管理', 0, 'app管理tab是否可见'),
	(15, '权限管理', 0, '权限管理tab是否可见'),
	(16, '活动管理', 0, '活动管理tab是否可见'),
	(101, '爆料列表', 1, '我的爆料/爆料列表可见权限'),
	(201, '优惠过期管理', 2, '优惠过期管理子tab是否可见'),
	(202, '过期提醒管理', 2, '过期提醒管理子tab是否可见'),
	(203, '发现频道管理', 2, '发现频道管理tab权限'),
	(204, '特卖频道管理', 2, '特卖频道管理tab权限'),
	(301, '小编自爆', 3, '小编自爆经验晒单'),
	(302, '编辑', 3, '编辑经验晒单'),
	(303, '删除', 3, '删除经验晒单'),
	(304, '发布', 3, '发布经验晒单'),
	(305, '精彩状态标签', 3, '设置经验晒单的精彩状态'),
	(10101, '查看/编辑', 101, '编辑未发布优惠'),
	(10102, '分配', 101, '分配未发布优惠'),
	(10103, '通过/拒绝', 101, '通过拒绝未发布优惠'),
	(10104, '发布', 101, '发布优惠'),
	(10105, '废弃', 101, '废弃优惠'),
	(10106, '小编自爆', 101, '爆料按钮－可见/不可见'),
	(20101, '置顶/高亮置顶', 201, '置顶高亮优惠'),
	(20102, '比对处理', 201, '比对处理按钮'),
	(20103, '查看/编辑', 201, '查看已发布优惠'),
	(20104, '废弃', 201, '废弃已发布优惠'),
	(20105, '售罄/过期标签', 201, '售罄/过期标签操作');


INSERT INTO `role` (`id`, `name`, `addtime`, `updatetime`) VALUES
	(1, 'admin', '2015-12-16 11:28:19', '2015-12-16 11:28:19'),
	(2, '经验晒单管理人', '2015-09-01 00:00:00', '2015-10-27 16:28:58'),
	(30, '超级管理员', '2015-09-01 00:00:00', '2015-09-01 00:00:00'),
	(42, '爆料管理人', '2015-11-17 09:53:18', '2015-11-17 09:53:18'),
	(44, 'app运营', '2015-11-18 16:27:41', '2015-11-18 16:27:41'),
	(45, '优惠发布人', '2015-11-18 16:28:18', '2015-11-18 16:28:18'),
	(46, '活动抽奖运营', '2015-11-18 16:40:53', '2015-11-18 16:40:53'),
	(49, '优惠审核人', '2015-11-20 16:22:02', '2015-11-20 16:22:02'),
	(50, '已发布爆料管理人', '2015-11-22 23:33:05', '2015-11-22 23:33:05'),
	(51, '查看爆料', '2015-11-23 14:35:18', '2015-11-23 14:35:18'),
	(52, '自爆', '2015-12-15 13:19:15', '2015-12-15 13:19:15'),
	(55, 'TestAddRolelyrg1', '2016-03-08 13:54:51', '2016-03-08 13:54:51'),
	(57, '特卖管理员', '2016-03-21 18:11:08', '2016-03-21 18:11:08'),
	(58, 'TestAddRolecnxcy', '2016-05-20 11:07:45', '2016-05-20 11:07:45');


INSERT INTO `role_auth` (`id`, `role_id`, `auth_id`) VALUES
	(80, 39, 201),
	(81, 39, 202),
	(209, 45, 1),
	(210, 45, 2),
	(211, 45, 101),
	(212, 45, 201),
	(213, 45, 202),
	(214, 45, 10101),
	(215, 45, 10102),
	(216, 45, 10103),
	(217, 45, 10104),
	(218, 45, 10105),
	(219, 45, 10106),
	(220, 45, 20101),
	(221, 45, 20102),
	(222, 45, 20103),
	(223, 45, 20104),
	(230, 44, 14),
	(231, 46, 12),
	(232, 46, 16),
	(267, 49, 1),
	(268, 49, 101),
	(269, 49, 10101),
	(270, 49, 10103),
	(271, 49, 10106),
	(272, 50, 2),
	(273, 50, 201),
	(274, 50, 20101),
	(275, 50, 20102),
	(276, 50, 20103),
	(277, 50, 20104),
	(284, 51, 1),
	(285, 51, 101),
	(286, 51, 10101),
	(322, 2, 16),
	(341, 52, 1),
	(342, 52, 101),
	(343, 52, 10106),
	(414, 42, 1),
	(415, 42, 101),
	(416, 42, 10101),
	(417, 42, 10103),
	(418, 42, 10105),
	(419, 42, 10106),
	(420, 42, 2),
	(421, 42, 201),
	(422, 42, 20101),
	(423, 42, 3),
	(424, 42, 301),
	(425, 42, 302),
	(572, 55, 201),
	(573, 55, 202),
	(575, 30, 1),
	(576, 30, 101),
	(577, 30, 10101),
	(578, 30, 10102),
	(579, 30, 10103),
	(580, 30, 10104),
	(581, 30, 10105),
	(582, 30, 10106),
	(583, 30, 2),
	(584, 30, 201),
	(585, 30, 20101),
	(586, 30, 20102),
	(587, 30, 20103),
	(588, 30, 20104),
	(589, 30, 20105),
	(590, 30, 202),
	(591, 30, 203),
	(592, 30, 3),
	(593, 30, 301),
	(594, 30, 302),
	(595, 30, 303),
	(596, 30, 304),
	(597, 30, 305),
	(598, 30, 4),
	(599, 30, 5),
	(600, 30, 6),
	(601, 30, 7),
	(602, 30, 8),
	(603, 30, 9),
	(604, 30, 10),
	(605, 30, 11),
	(606, 30, 12),
	(607, 30, 13),
	(608, 30, 14),
	(609, 30, 15),
	(610, 30, 16),
	(621, 57, 2),
	(622, 57, 201),
	(623, 57, 20101),
	(624, 57, 20102),
	(625, 57, 20103),
	(626, 57, 20104),
	(627, 57, 20105),
	(628, 57, 203),
	(629, 57, 204),
	(630, 1, 1),
	(631, 1, 101),
	(632, 1, 10101),
	(633, 1, 10102),
	(634, 1, 10103),
	(635, 1, 10104),
	(636, 1, 10105),
	(637, 1, 10106),
	(638, 1, 2),
	(639, 1, 201),
	(640, 1, 20101),
	(641, 1, 20102),
	(642, 1, 20103),
	(643, 1, 20104),
	(644, 1, 20105),
	(645, 1, 202),
	(646, 1, 203),
	(647, 1, 204),
	(648, 1, 3),
	(649, 1, 301),
	(650, 1, 302),
	(651, 1, 303),
	(652, 1, 304),
	(653, 1, 305),
	(654, 1, 4),
	(655, 1, 5),
	(656, 1, 6),
	(657, 1, 7),
	(658, 1, 8),
	(659, 1, 9),
	(660, 1, 10),
	(661, 1, 11),
	(662, 1, 12),
	(663, 1, 13),
	(664, 1, 14),
	(665, 1, 15),
	(666, 1, 16),
	(667, 58, 201),
	(668, 58, 202);
```


状态验证

  -对后台请求做filter，写入session，并请求验证当前用户是否是登录状态，并将用户基本信息写入session和context
  
权限树为静态数据，前后端都需要保存，并根据权限对应信息展示ui限制后端行为。

  -对每个操作行为接口，用注解方式表明对应的权限内容
  
  -注解中根据该用户的登录信息获取权限，并和注解权限对比。


在Filter中

```
  @Override
  publicvoiddoFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        if (httpServletRequest.getPathInfo().startsWith("/api/hui/inner/coupon/list")) {
            chain.doFilter(request, response);
        } else {
            CasClient casClient = new CasClient(httpServletRequest, httpServletResponse, casInfo);
            try {
                CasCheckResponse casCheckResponse = casClient.validateST();
                if (casCheckResponse == null) {
                    // unexpected response from uc
                    HttpResponseUtils.notLoginResponse(httpServletResponse);
                } else {
                    httpServletRequest.setAttribute(UCConstant.UC_USER_ATTRIBUTE_KEY,
                            casCheckResponse.getUcid());
                    httpServletRequest.setAttribute(UCConstant.UC_USERNAME_ATTRIBUTE_KEY,
                            casCheckResponse.getUsername());
                    httpServletRequest.setAttribute(UCConstant.UC_USER_AUTH_ATTRIBUTE_KEY, manager.findAuthIdsByUcId
                            (casCheckResponse.getUcid()));
                    chain.doFilter(request, response);
                }
            } catch (NotLoginException e) {
                HttpResponseUtils.notLoginResponse(httpServletResponse);
            } catch (McpackException e) {
                HttpResponseUtils.notLoginResponse(httpServletResponse);
            } catch (IOException e) {
                return;
            }
        }
  }
```

在AOP中

```
  @Around(value = "@annotation(casAuth)")
  public Object checkAuth(ProceedingJoinPoint joinPoint, CASAuth casAuth) throws Throwable {
        // check user attribute
        List<Long> authList = AuthUtils.getCurrentAuths();
        if (authList != null) {
            if (authList.contains(casAuth.authId())) {
                return joinPoint.proceed();
            } else {
                BaseResponse response = new BaseResponse(UCConstant.NOT_AUTH_STATUS, UCConstant.NOT_AUTH_MSG);
                return response;
            }
        } else {
            BaseResponse response = new BaseResponse(UCConstant.NOT_LOGIN_STATUS, UCConstant.NOT_LOGIN_MSG);
            return response;
        }
  }
```
AuthUtils.java

```
  publicstaticList<Long> getCurrentAuths() {
        HttpServletRequest httpServletRequest =
                ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        List<Long> auths = (List<Long>) httpServletRequest.getAttribute(UCConstant.UC_USER_AUTH_ATTRIBUTE_KEY);
        return auths;
  }
```

在web.xml中配置
<listener>
<listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
</listener>

问题QA
登陆账户是如何管理的，是不是每个UC账户都有登陆的权限?
是的，新的账户没有权限的话，登陆进去后停在一张啥都没有的页面
 
权限的分表
分为权限表，角色表，角色与权限的对照表，用户表，用户与角色的对照表
一个用户对应多个角色的时候，其权限是怎样的?
目前是将所有的权限做与操作
 
通过/hui-web-1.0/api/hui/inner/session?castk=6700fgd7f0c8ef980d159
获取到了用户的所有的权限
 
疑问
看到了从
CASStatusAOP.checkAuth(ProceedingJoinPoint joinPoint, CASAuth casAuth) throws Throwable
里面使用了前端传过来的权限的表做用户对一个Action是否有权限的判断，感觉这个表可以被前端更改的，是不是使用新读取出来的表比较合理些。
现在是在UCFilter里面加上了authlist，不是前端传输的
 
com.baidu.hui.web.filter.UCFilter过滤器过了权限
调用了UC的sdk实现是否登陆的判断
 
权限树在数据库中按照
Root->最大Tab->子Tab->子功能生成
Tab是否可见
 
前端是如何将authList告诉后端的？这个在header里面没有找到。
不是后端告诉的
 
Annotation的使用
注解
@Retention –什么时候使用该注解
@Target? –注解用于什么地方
@casAuth注解了方法的使用的地方
 
如何将不同的item分发给不同的user，怎么筛选出来的?
Item表有assign_id作为标示字段
 
这个API是干嘛用的? 哪里会调用?
/api/hui/inner/auth/user/verify
 
API role/add验证了Id和user name
那么uc那边，uc的名称会改变吗?如果改变的话，这里不久不对应了吗?
 
在分配Item的时候，没有验证user是否有优惠Tab权限?
这个pm可以接受，但是是一个可以优化的地方
 
 
帮助信息
Web.xml的加载过程
此段试图说明在
CASStatusAOP. checkAuth中调用了

```
  List<Long> authList = AuthUtils.getCurrentAuths();
    publicstaticList<Long> getCurrentAuths() {
        HttpServletRequest httpServletRequest =
                ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        List<Long> auths = (List<Long>) httpServletRequest.getAttribute(UCConstant.UC_USER_AUTH_ATTRIBUTE_KEY);
        return auths;
  }
```

这段代码中的RequestContextHolder的使用
 
启动Web项目的时候，容器包括(JBoss、Tomcat等)首先会读取项目web.xml配置文件的配置，当这一步没问题时，程序才启动起来.
1. 启动WEB项目时，容器首先会去它的配置文件web.xml读取两个节点:
<listener></listener>和<context-param></context-param>
2. 然后容器创建ServletContext（application）,这个容器的所有部分都共享这个上下文.
3. 容器以<context-param></context-param>的name作为键，value作为值，将其转化为键值对，存入ServletContext
4. 容器创建<listener></listener>中的类实例，根据配置的class类路径<listener-class>来创建监听，在监听中会有contextInitialized(ServletContextEvent args)初始化方法，启动Web应用时，系统调用Listener的该方法，在这个方法中获得：
ServletContext application =ServletContextEvent.getServletContext();
context-param的值= application.getInitParameter("context-param的键");
 
总的来说，web.xml的加载顺序是: context-param-> listener -> filter -> servlet

