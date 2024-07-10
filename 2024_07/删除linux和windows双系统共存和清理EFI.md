## 1. windows从grub中启动

[『香梨小课堂』误删windows的efi引导程序磁盘，如何通过grup2引导程序进入windows_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV15L411G7h1/?spm_id_from=333.337.search-card.all.click)


```cmd
insmod part_gpt  //让ls可以看到文件列表
ls
set root=(hd0,gpt1)
chainloader  /efi/Microsoft/Boot/bootmgfw.efi //如果root没选错的话，可以tab补齐，选错了话就换一换。
boot
```
## 2.删除efi分区中的deepin残留项 | 多系统共用系统引导。

[https://blog.csdn.net/qq_39740292/article/details/104983399](https://blog.csdn.net/qq_39740292/article/details/104983399)

使用到Diskpart,将efi区挂到一个没用的盘符。然后cmd进入删除多余操作系统的引导文件夹，就可以。



win+r 输入Diskpart

**disk选择** disk一般选择自己windows系统盘，准确得说是EFI所在盘，可能位置有windows系统盘，和以前装linux的系统盘。

**partition的选择** 类型选择系统类型。如果有多个系统类型，就一个个挂到盘符进去看是否有另外一个系统的EFI。

**盘符的选择** 不要选择已经创建过的盘符即可。

```cmd
list disk

select disk A

list partition

select partition B

assign letter=P
```

管理员启动cmd

```cmd
P:

cd EFI

rd /s deepin
```

Diskpart移除所设盘符

```cmd
remove letter=p
```



如果你用的是ubuntu，就删掉ubuntu，以此类推，只要删掉对应系统就可以，不要一键清空大法，那就真的喜提重装系统大礼包了。
