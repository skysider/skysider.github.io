---
layout: post
title: rop之return-to-dl-resolve
categories:
  - ctf
  - pwn
url: 416.html
id: 416
comments: false
date: 2017-05-15 10:16:12
tags:
---

可用工具: [roputils.py](https://github.com/inaz2/roputils) 适用场景：

*   无法leak内存
*   所有可以采用stack pivot的场景

### 一、（Partial RELRO 或者 no RELRO） and no PIE

#### 1\. 32位平台：

\_dl\_runtime\_resolve(link\_map *, reloc_offset)

利用roputils的dl\_resolve\_call(base, binsh\_addr)和dl\_resolve\_data(base, "system")完成system("/bin/sh")，注意：连续写入dl\_resolve\_call和dl\_resolve\_data的数据时，要将dl\_resolve\_call放在dl\_resolve_data的内容之前，否则sub esp会覆盖前面的数据

#### 2\. 64位平台

\_dl\_runtime\_resolve(link\_map *, reloc_index)

1.  无RELRO的情况下，可以修改.dynmaic段中的DT\_STRTAB指向.bss(注意要与.dynamic段的倒数第二项VERSYM保持至少0x180的间距），修改.dynamic段中的SYMTAB指向.bss，伪造sym，绕过sym->other检测，然后在STRTAB+sym->st\_name的位置写入"system"，参考 http://skysider.com/?p=602
2.  Patial RELRO
    1.  可以leak时，需要leak link\_map，令 link\_map +0x1c8 = 0（对应link\_map->l\_info\[49\]）（此时也可以leak 函数地址，然后确定libc版本，return-to-libc）
        
        if(\_\_builtin\_expect(ELFW(ST\_VISIBILITY)(sym->st\_other), 0) == 0)
        {
            const struct r\_found\_version *version = NULL;
            
            if (l->l\_info\[VERSYMIDX(DT\_VERSYM\])!=NULL)
            {
             const ElfW(Half) *vernum = 
               (const void *) D\_PTR(l, l\_info\[VERSYMIDX(DT_VERSYM)\]);
             ElfW(Half) ndx = vernum\[ELFW(R\_SYM)(reloc->r\_info)\] & 0x7ffff;
             version = &l->l_versions\[ndx\];
             if(version->hash) == 0
                 version = NULL;
            }
        
        在伪造sym之后造成VERSYM取值超出范围，造成segment fault
    2.  无leak(前提需要知道libc版本，需要有一个已经resolved过的函数）
        1.  修改link_map指针（GOT\[1\]，令其指向一个解析过的函数的GOT表项，比如.got\['read'\])
        2.  正确构造link\_map中的l\_info\[DT\_JMPREL\]、l\_info\[DT\_SYMTAB\]、l\_info\[DT\_STRTAB\]（可与原有的link\_map值相等）
        3.  伪造index, 使其指向伪造的 reloc
        4.  reloc->info>>0x20为SYMTAB的下标，使其指向伪造的Elf64_Sym项
        5.  if((sym->st_other) & 3 ) != 0) // 如果已经resolve过
                value = DL\_FIXUP\_MAKE\_VALUE(l, l->l\_addr + sym->st_value)
            
            令sym->st\_other的第2位为1，此时调用 DL\_FIXUP\_MAKE\_VALUE宏返回解析过的函数地址，正常情况下，link\_map的l\_addr=0(代表ELF文件中的虚拟地址与实际加载地址之间的差值）,  sym->st\_value为0，（got表项中的函数解析地址与实际虚拟地址之间的差值）。而通过伪造link\_map指针，使l->l\_addr为解析过的函数地址，sym->st\_value为system地址与该函数地址之间的差值，二者之和就是system的实际地址。解析后的system地址存放在reloc->offset + sym->st_value的地方，要确保可写

### 二、Full RELRO:

前提：可以leak内存 常用思路：读取.dynamic段中的d\_tag=DT\_DEBUG的项，其中的d\_value指向一个r\_debug结构体，定义如下：

struct r_debug { #64位版本
    int r_version;
    struct link\_map\_public *r_map;
    Elf64\_Addr r\_brk;
    enum {RT\_CONSISTENT, RT\_ADD, RT\_DELETE} r\_state;
    Elf64\_Addr r\_ldbase;

其中r\_map成员指向link\_map链表的头节点，头结点为当前elf文件的link\_map，伪造link\_map中l\_info\[DT\_JMPREL\]和l\_info\[DT\_STRTAB\]，伪造.rel.plt表以及.dynstr表，使rel表项指向已有的.dynsym项，由于.dynstr表被伪造，st\_name指向fake .dynstr中的字符串。 第二个问题是需要调用\_dl\_runtime\_resolve，但是开启了 Full Relro的程序 .plt.got表中没有该函数的地址，此时可以去遍历link\_map链表，通过link\_map->l\_name可以获取当前加载库的路径，从中可以找到未开启Full Relro的库文件（比如libc.so.6），读取link\_map->l\_info\[DT\_PLTGOT\]->d\_val可以找到.plt.got的地址，读取GOT\[2\]就可以找到\_dl\_runtime\_resolve的地址。 ![](http://www.inforsec.org/wp/wp-content/uploads/2016/01/Figure4-1024x650.png) 以上图片来自http://www.inforsec.org/wp/?p=389 附：

/\* Dynamic section tags.  */

#define DT_NULL		0
#define DT_NEEDED	1
#define DT_PLTRELSZ	2
#define DT_PLTGOT	3
#define DT_HASH		4
#define DT_STRTAB	5
#define DT_SYMTAB	6
#define DT_RELA		7
#define DT_RELASZ	8
#define DT_RELAENT	9
#define DT_STRSZ	10
#define DT_SYMENT	11
#define DT_INIT		12
#define DT_FINI		13
#define DT_SONAME	14
#define DT_RPATH	15
#define DT_SYMBOLIC	16
#define DT_REL		17
#define DT_RELSZ	18
#define DT_RELENT	19
#define DT_PLTREL	20
#define DT_DEBUG	21
#define DT_TEXTREL	22
#define DT_JMPREL	23
#define DT\_BIND\_NOW	24
#define DT\_INIT\_ARRAY	25
#define DT\_FINI\_ARRAY	26
#define DT\_INIT\_ARRAYSZ 27
#define DT\_FINI\_ARRAYSZ 28
#define DT_RUNPATH	29
#define DT_FLAGS	30
#define DT_ENCODING	32
#define DT\_PREINIT\_ARRAY   32
#define DT\_PREINIT\_ARRAYSZ 33

参考：

1.  http://angelboy.logdown.com/posts/283218-return-to-dl-resolve
2.  http://code.metager.de/source/xref/gnu/src/include/elf/common.h
3.  http://www.inforsec.org/wp/?p=389