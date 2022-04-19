```shell
find ./ -type d -name "Backup"  -exec rm -rf {} \;

sed -i "s/<cups\/cups.h>/\"cups\/cups.h\"/g"  `grep "cups/cups.h" -rl .`

find . -type f -size +100M  -print0 | xargs -0 du -h | sort -nr

cat *.txt | sort | uniq > test

#导出svn版本差异
for i in $(svn diff --summarize -r 248:276 svn://192.168.10.200/rootfs/ | awk '{ print $2 }'); do p=$(echo $i | sed -e 's{svn://192.168.10.200/rootfs/{{'); mkdir -p $(dirname $p); svn export $i $p; done 

#获取awk
./a.out |awk  'BEGIN{FS="[()]"} NR==5{print $2}'
 FS指定分隔符
 NR指定读取第几行输出
 
#查找文件权限
 find -type d -not -perm 775 -o -type f -not -perm 664
 
 
 sed -n '/192.xxx/'p xxxx |awk  'BEGIN{FS="[ ]"} {print $5 }'   |sort |uniq |wc -l
```

