## 配置用户名、邮箱

1.  `git config --global user.name "louisyanglu"`
2.  `git config --global user.email "418705294@qq.com"`

## 查看、设置SSH key

1.  查看key：![image-20210506231503468](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210506231612.png)

2.  本地生成key：`ssh-keygen -t rsa -C "418705294@qq.com"`，连按3次回车

3.  为github账号配置key（https://github.com/settings/keys）

    ![image-20210506232023790](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210506232028.png)

    粘贴`id_rsa.pub`中的内容

## git操作

1.  `git status`
2.  `git diff`
3.  `git add 文件`
4.  `git commit -m ""`
5.  `git push`