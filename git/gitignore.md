# 创建gitignore文件、git忽略上传文件

### 在git Hash
```
    touch .gitignore // 创建.gitignore文件
    vim .gitignore // 编辑.gitignore文件
    # 在.gitignore文件中是注释
    
    .idea # 忽略idea文件夹
    *.md #忽略所有md结尾的文件
    !README.md  #但是README.md除外
    
    /node_modules #仅忽略根目录下的node_modules文件夹
    
    dist/  #忽略dist文件夹下的所有文件
    
```