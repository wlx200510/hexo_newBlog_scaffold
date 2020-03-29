# 博客部署步骤

本博客不使用github，使用腾讯云的机器进行部署，假设腾讯云的机器地址为 123.123.122.66

在增加完新文章之后：

```shell
npm run build
tar -zcvf ./public.tar.gz ./public
scp ./public.tar.gz root@123.123.122.66:/root/
# 以下是登录腾讯云主机的操作
ssh 123.123.122.66
rm -rf my_blog && tar -zxvf public.tar.gz
mv public my_blog
rm -rf public.tar.gz
```

后续考虑做到`hexo d`的命令中