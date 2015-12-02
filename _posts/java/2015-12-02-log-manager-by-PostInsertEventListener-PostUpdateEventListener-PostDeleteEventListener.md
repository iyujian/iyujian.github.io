---
layout: post
title:  "使用 Hibernate 的 PostInsertEventListener, PostUpdateEventListener, PostDeleteEventListener 做日志管理"
date:   2015-12-02 13:04:25 +0800
categories: java
tags: hibernate
---
Hibernate 为我们提供了事件监听机制，我们只需要实现相应的监听器，就可以追踪到对象的修改，比如 PostInsertEventListener, PostUpdateEventListener, PostDeleteEventListener、PostLoadEventListener等等，都在 org.hibernate.event.spi 包中。

<!-- more -->

1、实现 PostInsertEventListener, PostUpdateEventListener, PostDeleteEventListener

{% highlight java linenos %}
package com.***.listener;

import com.***.entity.LogEntity;
import com.***.enums.LogType;
import com.***.manager.LogManager;
import com.***.utils.SysUserUtil;
import com.***.UserDto;
import org.hibernate.event.spi.*;
import org.hibernate.persister.entity.EntityPersister;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.sql.Timestamp;
import java.text.SimpleDateFormat;
import java.util.Date;

@Component
public class LogListener implements PostInsertEventListener, PostUpdateEventListener, PostDeleteEventListener {

    @Autowired
    private LogManager logManager;

    @Override
    public void onPostDelete(PostDeleteEvent event) {

        UserDto user = SysUserUtil.getCurrentUser();
        if(null != user) {
            LogEntity log = new LogEntity();
            log.setOperatorTime(new Date());
            log.setObjectId((Long) event.getId());
            log.setObjectName(event.getEntity().getClass().getSimpleName());
            log.setEvent("delete");
            log.setRemark("删除");
            log.setOperatorName(user.getUsername());
            log.setOperatorId(user.getId());
            log.setType(LogType.MANAGER.getCode());
            logManager.save(log);
        }

    }

    @Override
    public void onPostInsert(PostInsertEvent event) {

        UserDto user = SysUserUtil.getCurrentUser();
        if(null != user && !"LogEntity".equals(event.getEntity().getClass().getSimpleName())) {
            LogEntity log = new LogEntity();
            log.setOperatorTime(new Date());
            log.setObjectId((Long) event.getId());
            log.setObjectName(event.getEntity().getClass().getSimpleName());
            log.setEvent("insert");
            log.setRemark("新增");
            log.setOperatorName(user.getUsername());
            log.setOperatorId(user.getId());
            log.setType(LogType.MANAGER.getCode());
            logManager.save(log);
        }

    }

    /**
     * 当对属性进行更新操作时，记录日志，同时备注发生变更的字段以及变更前后的值
     * @param event
     */
    @Override
    public void onPostUpdate(PostUpdateEvent event) {
        UserDto user = SysUserUtil.getCurrentUser();
        if(null != user) {

            Object[] oldState = event.getOldState();
            Object[] newState = event.getState();
            if(!oldState.equals(newState)) {
                LogEntity log = new LogEntity();
                log.setOperatorTime(new Date());
                log.setObjectId((Long) event.getId());
                log.setObjectName(event.getEntity().getClass().getSimpleName());
                log.setEvent("update");
                log.setOperatorName(user.getUsername());
                log.setOperatorId(user.getId());
                log.setType(LogType.MANAGER.getCode());

                StringBuilder remark = new StringBuilder("");

                for(int i=0, length=oldState.length; i<length; i++) {
                    Object oldValue = oldState[i];
                    Object newValue = newState[i];

                    if(!(oldValue == null && newValue == null)) {

                        if(isDate(newValue)) {
                            newValue = formatDate(newValue);
                        }
                        if(isDate((oldValue))) {
                            oldValue = formatDate(oldValue);
                        }

                        if(oldValue == null || newValue == null) {
                            remark.append(event.getPersister().getPropertyNames()[i]+":"+oldValue+" -> "+newValue+";");
                        } else {
                            if(!oldValue.equals(newValue)) {
                                remark.append(event.getPersister().getPropertyNames()[i]+":"+oldValue+" -> "+newValue+";");
                            }
                        }
                    }

                }

                log.setRemark(remark.toString());
                logManager.save(log);
            }

        }
    }

    @Override
    public boolean requiresPostCommitHanding(EntityPersister persister) {
        return false;
    }

    private boolean isDate(Object date) {

        if(date == null) {
            return false;
        } else {
            if(date instanceof Date || date instanceof java.sql.Date || date instanceof Timestamp) {
                return true;
            }
        }
        return false;
    }

    private String formatDate(Object date) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateStr = "";
        if(date instanceof Date) {
            dateStr = sdf.format(date);
        } else if(date instanceof java.sql.Date) {
            dateStr = sdf.format(new Date(((java.sql.Date) date).getTime()));
        } else if(date instanceof Timestamp) {
            dateStr = sdf.format(new Date(((Timestamp) date).getTime()));
        } else {
            dateStr = String.valueOf(date);
        }
        return dateStr;
    }
}
{% endhighlight %}

2、注册监听器，可以在spring容器启动后进行注册（[在Spring启动后执行一些操作](execute-after-spring-startup.html "在Spring启动后执行一些操作")）。

{% highlight java linenos %}
/** 获取SessionFactory的方式：1、直接注入SessionFactory；2、如果用的是 JPA 的话，那么需要通过entityManagerFactory来得到SessionFactory
*/
EventListenerRegistry registry  = ((SessionFactoryImpl)entityManagerFactory.unwrap(SessionFactory.class)).getServiceRegistry().getService(EventListenerRegistry.class);
registry.getEventListenerGroup(EventType.POST_INSERT).appendListener(logListener);
registry.getEventListenerGroup(EventType.POST_UPDATE).appendListener(logListener);
registry.getEventListenerGroup(EventType.POST_DELETE).appendListener(logListener);
{% endhighlight %}