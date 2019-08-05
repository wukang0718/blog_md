# 基础sql语句

> 删除数据库
 ```
    DROP DATABASE IF EXISTS database;
 ```
 
 > 创建数据库
 ```
    CREATE DATABASE database charset=utf8;
 ```
 
 > 进入数据库
 ```
    USE database; 
 ```
 
 > 创建表格
 ```
    CREATE TABLE info (
        id INT PRIMARY KEY,
        title VARCHAR(128) NOT NULL,
        contentPath VARCHAR(128) NOT NULL, -- 内容path
        fabulous INT DEFAULT 0, -- 点赞数
        userId INT NOT NULL,
        createDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- 创建时间，默认当前时间
        FOREIGN KEY(userId) REFERENCES userInfo(id) -- 外键约束
    );
 ```