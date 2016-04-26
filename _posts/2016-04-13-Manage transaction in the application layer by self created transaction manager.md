---
layout: post
title: Manage transaction in the application layer by self created transaction manager
comments: false
author: "Yin Haomin"
date: 2016-04-13 01:00:00
tags:
    - Spring
    - Transaction
    - Application layer transaction
    - Transaction manager
---

如何在应用层实现transaction manager管理事务，举个例子我们需要在创建一个订单的时候，对于优惠券，库存进行操作。一旦出现异常就需要对整个操作进行回滚。

以下实现了一个业务层事务的代码: [业务层事务](https://github.com/yinhaomin/common-test/tree/master/common-test-base/src/main/java/com/baidu/common/test/base/transaction)

以下为一个实现的实例:

首先实现一个TransactionOperation

```
@Log4j
@Data
@NoArgsConstructor
public class AddSubscribe2PassportOperation implements TransactionOperation {

    /**
     * 订阅的passportId
     */
    private Long passportId;

    /**
     * 订阅的Tags
     */
    private List<TagBo> tagList;

    /**
     * 订阅的Service
     */
    private SubscribeService subscribeService;

    /**
     * 所有参数的Constructor.
     * 
     * @param passportId
     * @param passportTags
     * @param subscribeService
     */
    public AddSubscribe2PassportOperation(Long passportId, List<TagBo> tagList, SubscribeService subscribeService) {
        super();
        this.passportId = passportId;
        this.tagList = tagList;
        this.subscribeService = subscribeService;
    }

    @Override
    public String getName() {
        String clazzName = Thread.currentThread().getStackTrace()[1].getClassName();
        return clazzName;
    }

    /**
     * 添加订阅的Tag，添加失败会Throw exception，过程中出现Exception会Throw exception.
     */
    @Override
    public Object execute() {
        try {
            if (this.passportId != null && this.passportId != 0) {
                if (CollectionUtils.isNotEmpty(tagList)) {
                    List<PassportSubscribeTag> passportTags = Lists.newArrayList();
                    for (TagBo tagBo : tagList) {
                        PassportSubscribeTag passportTag = new PassportSubscribeTag();
                        passportTag.setPassportId(passportId);
                        passportTag.setTagId(tagBo.getId());
                        passportTag.setTagName(tagBo.getName());
                        passportTags.add(passportTag);
                    }
                    int result = subscribeService.insertPassportTag(passportTags);
                    if (result != tagList.size()) {
                        String message = "The affected tags size is not equal to the inputed size.";
                        log.error(message);
                        throw new TransactionException(message);
                    }
                }
            }
        } catch (Exception e) {
            StringBuilder message = new StringBuilder();
            message.append("Captured exception:").append(e);
            log.error("Captured exception:", e);
            throw new TransactionException(message.toString());
        }
        return null;
    }

    /**
     * 回滚相应的操作.
     */
    @Override
    public void rollback(TransactionContext transactionContext) {
        try {
            if (this.passportId != null && this.passportId != 0) {
                if (CollectionUtils.isNotEmpty(this.tagList)) {
                    List<String> subscribeNames = Lists.newArrayList();
                    for (TagBo tagBo : this.tagList) {
                        subscribeNames.add(tagBo.getName());
                    }
                    subscribeService.deleteSubscribedTagsByPassportId(passportId, subscribeNames);
                }
            }
        } catch (Exception e) {
            StringBuilder message = new StringBuilder();
            message.append("Captured exception:").append(e);
            log.error("Captured exception:", e);
            throw new TransactionException(message.toString());
        }
    }
}
```

以下为使用的方式

```
        // 获取当前的Method的名称.
        String methodName = Thread.currentThread().getStackTrace()[1].getMethodName();
        TransactionStatus transactionStatus = applicationTransactionManager.execute(methodName,
                new TransactionCallback() {

                    @Override
                    public void doInTransaction(Transaction transaction) {
                        // 为passport写入订阅的Tag
                        transaction.execute(new AddSubscribe2PassportOperation(passportId, subscriptionAdd.getTags(),
                                subscribeService));

                        // 为device写入订阅Tag
                        transaction.execute(new AddSubscribe2DeviceOperation(subscriptionAdd.getDeviceId(),
                                subscriptionAdd.getTags(), subscribeService));
                    }
                });
        if (transactionStatus.getStatus() == TransactionStatus.SUCCESS) {
            return BaseResponse.SUCCESS;
        } else {
            return BaseResponse.ERROR;
        }
```

