1. 安装mercurial，拉取源代码
```sh
sudo apt-get install mercurial
hg clone http://hg.openjdk.java.net/jdk8u/jdk8u/
```

abort: stream ended unexpectedly (got 189475 bytes, expected 205870)

```sh
hg init jdk8u
#rev: 最新版本，见http://hg.openjdk.java.net/jdk8u/jdk8u/
#step: 每次拉取变更数
rev=2436; declare -i step; step=200; declare -i i; for ((i=0; i<$rev;)); do if (($i+$step<$rev)); then i+=$step; else i=$rev; fi; hg pull -r ${i} http://hg.openjdk.java.net/jdk8u/jdk8u/; done; unset step; unset i; unset rev;
hg update
chmod 755 get_source.sh
./get_source.sh
```

ERROR: Need initial repository to use this script
```sh
cd .hg && touch hgrc
cd .. && ./get_source.sh
```

ERROR: Need initial clone with 'hg paths default' defined
```sh
sh -c 'echo "[paths]
default = http://hg.openjdk.java.net/jdk8u/jdk8u/
">.hg/hgrc'
./get_source.sh
# corba、jaxp、jaxws在Java11已删除，nashorn在Java11也被废弃
# corba			1198 files
# hotspot		4846 files
# jaxp			2072 files	
# jaxws			3735 files
# jdk			23848 files
# langtools		6396 files
# nashorn		2774 files
```

2. 迁移到git

```sh
git clone https://github.com/frej/fast-export.git
hg log jdk8u | grep user: | sort | uniq | sed 's/user: *//' > authors
# mercurial只有author，缺少邮箱信息，需要补上
cat authors  |awk '{print "\"" $1 "\"=\"" $1 " <" $1 "@openjdk.java.net>" "\""}'>authors.map
mkdir openjdk
cd openjdk
# Windows平台需要设置成忽略大小写
git config core.ignoreCase false
# mercurial有forest的概念，需先迁移子仓库
forest=("corba" "hotspot" "jaxp" "jaxws" "jdk" "langtools" "nashorn");for t in ${forest[@]}; do git init $t;cd $t;~/fast-export/hg-fast-export.sh -r ~/jdk8u/$t -A ~/authors.map --hgtags;cd ..;done;unset forest;
# 创建mercurial子仓库和git子模块的映射关系文件。
cat > submodules.map<<EOF
"corba"="../corba"
"hotspot"="../hotspot"
"jaxp"="../jaxp"
"jaxws"="../jaxws"
"jdk"="../jdk"
"langtools"="../langtools"
"nashorn"="../nashorn"
EOF
git init jdk8u && cd jdk8u
~/fast-export/hg-fast-export.sh -r ~/jdk8u -A ~/authors.map --subrepo-map=../submodules.map  --hgtags
# 如果未能成功生成.gitmodules，可以在主项目先checkout，再执行"forest=("corba" "hotspot" "jaxp" "jaxws" "jdk" "langtools" "nashorn");for t in ${forest[@]}; do git submodule add ../$t $t;done;git commit -a -m "submodules";unset forest;"
git remote add origin http://vcs.iamwhatiam.ml/iMinusMinus/openjdk.git
git push -u origin master --recurse-submodules=check
git push --tags
```

3. hotspot概览

```
|--agent									#Serviceability Agent是一个运行在独立进程的hotspot诊断工具，此目录包含c++和java代码
|--make										#构建hotspot的配置文件
|--src
|  |--cpu
|  |--os
|  |--os_cpu
|  └-share
|    |--tools
|	 |  |--hsdis
|	 |  |--IdeaGraphVisualizer
|	 |  |--LogCompilation
|	 |  |--ProjectCreator
|	 |  └-*.java
|	 └-vm
|	    |--adlc								#Architecture Description Language Compiler，将os和os_cpu下ad文件编译成c++文件
|		|--asm
|		|--c1								#c1 compiler，即client编译器
|		|--ci
|		|--classfile						#类文件处理，如类加载、系统符号表
|		|--code
|		|--complier
|		|--gc_implementation				#GC实现，包含CMS、G1、ParallelScanvenge、ParNew及共享的代码
|		|--gc_interface						#GC接口，包含GC名称定义、引发GC的原因定义、内存分配、垃圾回收
|		|--interpreter						#字节码解释器
|		|--libadt							#Abstract Data Type
|		|--memory							#内存管理及旧的分代垃圾回收
|		|--oops								#Hotpost对java对象系统的实现
|		|--opto								#c2 compiler，即server编译器
|		|--precompiled						#预编译头文件
|		|--prims							#JVMTI等Hotpost VM对外接口
|		|--runtime							#运行时库，包含锁、线程等
|		|--services							#支持JMX之类管理功能的接口
|		|--shark							#基于LLVM的JIT编译器
|		|--trace
|		└-utilities
|		
└-test
```

参考：
1. [migrating to git](https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git)
2. [fast-export submodules](https://github.com/frej/fast-export/blob/master/README-SUBMODULES.md)
3. [AdoptOpenJDK](https://github.com/AdoptOpenJDK/openjdk-jdk8u)
