# Shell 模板
[TOC]

## 得到当前路径
```shell
CWD=$(cd "$(dirname "$0")"; pwd)
```

## usage parse_params
```shell
function usage() {
    echo "Usage:`basename $0` -f csv_file"
}

function parse_params(){
    if [ "$#" -lt 1 ];then
        #echo "Error: No input parameters!"
        usage
        exit 1
    fi

    while getopts "f:" optname
    do
        case "$optname" in
            "f")
                csv_file=$OPTARG
                if [ ! -e "$csv_file" ];then
                    echo "${csv_file} doesn't exist!"
                    exit 1
                fi
                ;;
             *)
                echo "Unknown parameter"
                ;;
        esac
    done
}
```
