# nodejs连接mysql

 > 执行npm install mysql --save
 
 ```
    const mysql = require("mysql");
    
    const connection = mysql.createConnection({
        host: "localhost", // 主机名
        port: 3306, // 端口号，默认3306
        user: "root", // 用户名
        password: "", // 密码
        database: "database" // 数据库名称
    });
    
    connection.connect();
    
    function request(sql, params) {
        return new Promise((resolve, reject) => {
            connection.query(sql, params, (err, result, fields) => {
                if (!err) {
                    resolve(result)
                } else {
                    reject(err)
                }
            })
        })
    
    }
    
    module.exports = request;
 ```