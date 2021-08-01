作为一名运维，我们经常会需要编写脚本来完成一些自动化工作，这里提供一个shell脚本的模板

```
#!/bin/sh
################ Version Info ##################
# Create Date: 2021-05-26
# Author:      vishon
# Mail:        nwx_qdlg@163.com
# Version:     1.0
# Attention:   shell脚本模板
################################################

# 加载环境变量 
# 如果脚本放到crontab中执行，会缺少环境变量，所以需要添加以下3行
. /etc/profile
. ~/.bash_profile
. /etc/bashrc

# 脚本所在目录即脚本名称
script_dir=$( cd "$( dirname "$0"  )" && pwd )
script_name=$(basename ${0})
# 日志目录
log_dir="${script_dir}/log"
[ ! -d ${log_dir} ] && {
   mkdir -p ${log_dir}
}
 
errorMsg(){
   echo "USAGE:$0 arg1 arg2 arg3"
   exit 2
}
 

doCode() {
   echo $1
   echo $2
   echo $3
}
 
main() {
   if [ $# -ne 3 ];then
     errorMsg
   fi
   doCode "$1" "$2" "$3"
}

# 需要把隐号加上，不然传入的参数就不能有空格
main "$@"
```