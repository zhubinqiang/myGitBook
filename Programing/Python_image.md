# Python 计算机视觉编程
[TOC]

## 安装PIL类库
```sh
## 这个不一定能安装
pip install PIL

pip install Pillow
```

## 安装matplotlib
```sh
pip install matplotlib==2.2.3
```


```py
from PIL import Image

def main():
    im = Image.open('vim.gif')
    w, h = im.size
    ## 缩略图
    im.thumbnail((w/2, h/2))
    im.save('out.gif')
    
if __name__ == '__main__':
    main()
```


