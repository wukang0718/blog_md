# Eslint关闭文件校验

1. 关闭段落校验
    ```
    /* eslint-disable */
        代码块
    /* eslint-enable */
    ```
2. 关闭当前行校验
    ```
    一行代码 // eslint-disable-line
    ```
3. 关闭下一行校验
    ```
    // eslint-disable-next-line
    下一行的代码
    ```
4. 关闭对这单一文件的校验
    >在文件头部加上注释,eslint在校验的时候会跳过后续的代码
    ```
    /* eslint-disable */
    ```