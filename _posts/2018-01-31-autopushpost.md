
```
s="cyxpost"
s=$1
sign=0
DATE=$(date +%Y"-"%m"-"%d)
postname=$DATE-$s".md"
echo $postname
for file in ./cuiyixiao.github.io/_posts/*
do
    if [[ $file =~ $s ]]
    then
        vimx $file
        mv $file ./cuiyixiao.github.io/_posts/$postname
        sign=1
    fi
done
if [ $sign == 0 ]
then
    vimx ./cuiyixiao.github.io/_posts/$postname
fi
s3="y"
read -p 'is push?[y/n]' s1
if [[ $s1 == $s3 ]]
then
    cd ./cuiyixiao.github.io
    git add --all
    git commit -m "update"
    git push -u origin
    cd ..
fi
```

- 用于提交博客文章，自动命名XXXX-XX-XX-postname.md，
- 如果有同名文件，则打开相应文件更改相应的日期。
- vim保存完后会提示是否上传，上传至github
