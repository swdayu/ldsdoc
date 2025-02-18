链接选项
=========

* `GNU LD简介`_
* `选项汇总`_

GNU LD简介
----------

ld 将多个目标文件和库文件组合在一起，重新定位它们的数据，并将符号引用连接起来。通常，编
译程序的最后一步是运行 ld。ld 接受用 AT&T 链接器命令语言语法（超集）编写的链接器命令语
言文件，以提供对链接过程的明确和完全控制。

这个版本的 ld 使用通用的 BFD 库来操作目标文件。这使得 ld 能够读取、组合和写入许多不同
格式的目标文件，例如 COFF 或 a.out。不同格式可以链接在一起，以产生任何可用类型的目标文
件。除了其灵活性外，GNU 链接器在提供诊断信息方面比其他链接器更有帮助。许多链接器在遇到
错误时会立即停止执行，而 ld 只要可能会继续执行，允许你识别其他错误（或者在某些情况下，
尽管存在错误，也能得到输出文件）。

你可以通过环境变量 GNUTARGET、LDEMULATION 和 COLLECT_NO_DEMANGLE 来改变 ld 的行为。
GNUTARGET 确定输入文件目标格式，如果你不使用 ‘-b’（或其同义词 ‘--format’）。它的值应
该是输入格式的 BFD 名称之一。如果环境中没有 GNUTARGET，ld 使用目标平台的自然格式。如果
GNUTARGET 设置为 default，那么 BFD 将尝试通过检查二进制输入文件来发现输入格式；这种方
法通常会成功，但由于没有方法确保用于指定目标文件格式的魔数是唯一的，因此可能存在潜在的歧
义。然而，每个系统上 BFD 的配置程序将该系统的传统格式放在搜索列表的首位，因此歧义会以传
统方式解决。

LDEMULATION 确定默认仿真，如果你不使用 ‘-m’ 选项。仿真可以影响链接器行为的各个方面，特
别是默认链接器脚本。你可以使用 ‘--verbose’ 或 ‘-V’ 选项列出可用的仿真。如果未使用
‘-m’ 选项，且 LDEMULATION 环境变量未定义，则默认仿真取决于链接器的配置方式。

通常，链接器将默认对符号进行反混淆。然而，如果环境中设置了 COLLECT_NO_DEMANGLE，那么它
将默认不对符号进行反混淆。这个环境变量也以类似的方式被 gcc 链接器包装程序使用。默认行为
可以通过 ‘--demangle’ 和 ‘--no-demangle’ 选项覆盖。

GNU 链接器 ld 旨在涵盖广泛的情况，并尽可能与其他链接器兼容。因此，你有许多选择来控制其
行为。链接器支持大量的命令行选项，但在实际使用中，很少有选项用于任何特定的上下文中。例
如，ld 的一个常见用途是在标准、受支持的 Unix 系统上链接标准 Unix 目标文件。在这样的系
统上，要链接一个名为 hello.o 的文件：ld -o output /lib/crt0.o hello.o -lc。

这告诉 ld 将文件 /lib/crt0.o 与 hello.o 以及来自标准搜索目录的库 libc.a 链接，生成一
个名为 output 的文件。一些命令行选项中可以指定在命令行的任何位置。然而，涉及文件的选
项，如 ‘-l’ 或 ‘-T’，会在命令行中出现的位置（相对于目标文件和其他文件选项）读取文件。
重复的带有不同参数的非文件选项，要么没有进一步的效果，要么会覆盖该选项的前面出现的值（即
命令行上更靠左的位置）。在下面的描述中，会注明可以有意义多次指定的选项。

非选项参数是要链接在一起的目标文件或库文件。它们可以跟在、先于或与命令行选项混合在一起，
但目标文件不能放在选项和这个选项的参数之间。通常，链接器至少被调用处理一个目标文件，但你
可以使用 ‘-l’、‘-R’ 和脚本命令语言指定其他形式的二进制输入文件。如果没有指定任何二进制
输入文件，链接器不会产生任何输出，并发出 ‘No input files’ 的消息。

如果链接器无法识别目标文件的格式，它将假设它是一个链接器脚本。以这种方式指定的脚本会用于
增强主链接器脚本（无论是默认链接器脚本还是使用 ‘-T’ 指定的脚本）。此功能允许链接器链接
到一个看起来像是目标文件或库文件，但实际上只是定义了一些符号值，或者使用 INPUT 或 
GROUP 加载其他对象的文件。以这种方式指定脚本只是增强了主链接器脚本，额外的命令放在主脚
本之后；使用 ‘-T’ 选项完全替换默认链接器脚本，但请注意 INSERT 命令的效果。参见链接脚
本部分。

对于名称为单个字母的选项，选项参数要么紧跟在选项字母后面中间不能有空格，要么作为单独的参
数跟随在需要它的选项之后。对于名称为多个字母的选项，选项名称前面可以是一个破折号或两个破
折号；例如，‘-trace-symbol’ 和 ‘--trace-symbol’ 是等价的。注意这条规则有一个例外，以
小写字母 ‘o’ 开头的多个字母选项只能由两个破折号引导。这是为了减少与 ‘-o’ 选项的混淆。
所以，例如，‘-omagic’ 将输出文件名设置为 ‘magic’，而 ‘--omagic’ 则在输出上设置
NMAGIC 标志。多个字母选项的参数必须要么与选项名称用等号分隔，要么作为单独的参数跟随在需
要它们的选项之后。例如，‘--trace-symbol foo’ 和 ‘--trace-symbol=foo’ 是等价的。多个
字母选项名称的唯一缩写是被接受的。

注意，如果链接器是通过编译器驱动程序（例如 gcc）间接调用的，那么所有链接器命令行选项都
应以 ‘-Wl,’ 为前缀，例如：gcc -Wl,--start-group foo.o bar.o -Wl,--end-group。这很
重要，因为否则编译器驱动程序可能会默默丢弃链接器选项，导致链接失败。当通过驱动程序传递需
要值的选项时，也可能引起混淆，因为选项和参数之间的空格会作为分隔符，导致驱动程序只将选项
传递给链接器，而将参数传递给编译器。在这种情况下，使用单个字母和多个字母选项的连接形式最
为简单，例如：gcc foo.o bar.o -Wl,-eENTRY -Wl,-Map=a.map。

选项汇总
--------

链接选项汇总： ::

    @file --defsym=symbol=expression
    -Ifile --dynamic-linker=file --no-dynamic-linker
    -a keyword
    --audit AUDITLIB
    -b input-format --format=input-format
    -c MRI-commandfile --mri-script=MRI-commandfile
    -d -dc -dp
    --depaudit AUDITLIB -P AUDITLIB
    --enable-linker-version --disable-linker-version
    --enable-non-contiguous-regions --enable-non-contiguous-regions-warnings
    -e entry --entry=entry
    --exclude-libs lib,lib,...
    --exclude-modules-for-implib module,module,...
    -E --export-dynamic --no-export-dynamic
    --export-dynamic-symbol=glob --export-dynamic-symbol-list=file
    -EB -EL
    -f name --auxiliary=name -F name --filter=name -fini=name
    -g -G value --gpsize=value
    -h name -soname=name
    -i -init=name
    -l namespec --library=namespec
    -L searchdir --library-path=searchdir
    -m emulation
    --remap-inputs=‘pattern’=‘filename’ --remap-inputs-file=‘file’
    -M --print-map
    --print-map-discarded --no-print-map-discarded
    --print-map-locals --no-print-map-locals
    -n --nmagic -N --omagic --no-omagic
    -o output --output=output --dependency-file=depfile
    -O level -plugin name
    --push-state --pop-state
    -q --emit-relocs --force-dynamic
    -r --relocatable -R filename --just-symbols=filename
    --rosegment --no-rosegment
    -s --strip-all -S --strip-debug --strip-discarded --no-strip-discarded
    -plugin-save-temps
    -t --trace -T scriptfile --script=scriptfile
    -dT scriptfile --default-script=scriptfile
    -u symbol --undefined=symbol
    --require-defined=symbol
    -Ur --orphan-handling=MODE --unique[=SECTION]
    -v --version -V
    -x --discard-all -X --discard-locals
    -y symbol --trace-symbol=symbol -Y path -z keyword
    -( archives -) --start-group archives --end-group
    --accept-unknown-input-arch --no-accept-unknown-input-arch
    --as-needed --no-as-needed --add-needed --no-add-needed
    -assert keyword
    -Bdynamic -dy -call_shared
    -Bgroup -Bstatic -dn -non_shared -static
    -Bsymbolic -Bsymbolic-functions -Bno-symbolic
    --dynamic-list=dynamic-list-file --dynamic-list-data
    --dynamic-list-cpp-new --dynamic-list-cpp-typeinfo
    --check-sections --no-check-sections
    --copy-dt-needed-entries --no-copy-dt-needed-entries
    --cref --ctf-variables --no-ctf-variables --ctf-share-types=method
    --no-define-common
    --force-group-allocation
    --demangle[=style] --no-demangle
    --embedded-relocs
    --disable-multiple-abs-defs
    --fatal-warnings --no-fatal-warnings
    -w --no-warnings
    --force-exe-suffix
    --gc-sections --no-gc-sections --print-gc-sections
    --no-print-gc-sections --gc-keep-exported
    --print-output-format --print-memory-usage
    --help --target-help
    -Map=mapfile
    --no-keep-memory --no-undefined -z defs
    --allow-multiple-definition -z muldefs
    --allow-shlib-undefined --no-allow-shlib-undefined
    --error-handling-script=scriptname
    --no-undefined-version
    --default-symver --default-imported-symver
    --no-warn-mismatch --no-warn-search-mismatch
    --whole-archive --no-whole-archive --noinhibit-exec
    -nostdlib --oformat=output-format --out-implib file
    -pie --pic-executable -no-pie -qmagic -Qy
    --relax --no-relax
    --retain-symbols-file=filename
    -rpath=dir -rpath-link=dir
    --section-ordering-file=script
    -shared -Bshareable
    --sort-common --sort-common=ascending --sort-common=descending
    --sort-section=name --sort-section=alignment
    --spare-dynamic-tags=count
    --split-by-file[=size] --split-by-reloc[=count]
    --stats --sysroot=directory --task-link --traditional-format
    --section-start=sectionname=org -Tbss=org -Tdata=org -Ttext=org
    -Ttext-segment=org -Trodata-segment=org -Tldata-segment=org
    --unresolved-symbols=method --dll-verbose
    --verbose[=NUMBER] --version-script=version-scriptfile
    --warn-common --warn-constructors
    --warn-execstack --warn-execstack-objects --no-warn-execstack
    --error-execstack --no-error-execstack
    --warn-multiple-gp --warn-once
    --warn-rwx-segments --no-warn-rwx-segments
    --error-rwx-segments --no-error-rwx-segments
    --warn-section-align --warn-textrel --warn-alternate-em
    --warn-unresolved-symbols --error-unresolved-symbols
    --wrap=symbol
    --eh-frame-hdr --no-eh-frame-hdr
    --no-ld-generated-unwind-info
    --enable-new-dtags --disable-new-dtags
    --hash-size=number --hash-style=style
    --compress-debug-sections=none|zlib|zlib-gnu|zlib-gabi|zstd
    --reduce-memory-overheads
    --max-cache-size=size
    --build-id --build-id=style
    --package-metadata=JSON

**@file** ::

    从文件中读取命令行选项。读取到的选项会插入到原 `@file` 选项所在的位置。如果文件不
    存在或无法读取，那么该选项将按字面意思处理，不会被移除。

    文件中的选项由空白字符分隔。可以通过将整个选项用单引号或双引号括起来，从而在选项中
    包含空白字符。任何字符（包括反斜杠）都可以通过在其前面加上反斜杠的方式包含在选项
    中。文件本身可能包含额外的 `@file` 选项；任何此类选项都将被递归处理。

**-h name -soname=name** ::

    在创建 ELF 共享对象时，将内部的 DT_SONAME 字段设置为指定的名称。当一个可执行文件是
    与具有 DT_SONAME 字段的共享对象进行链接的，那么在运行该可执行文件时，动态链接器将
    尝试加载 DT_SONAME 字段指定的共享对象，而不是使用传递给链接器的文件名。

**-x --discard-all** ::

    删除所有本地符号。

**-X --discard-locals** ::

    删除所有临时本地符号，这些符号以特定于系统的本地标签前缀开头，对于 ELF 系统通常是
    .L，对于传统的 a.out 系统通常是 L。

**--gc-sections --no-gc-sections** ::

    启用对未使用的输入节的垃圾回收。对于不支持此选项的目标平台，该选项将被忽略。通过在
    命令行上指定 --no-gc-sections，可以恢复默认行为（即不执行此垃圾回收）。请注意，支
    持对 COFF 和 PE 格式目标进行垃圾回收，但目前该实现被认为是实验性的。

    --gc-sections 通过检查符号和重定位信息来确定哪些输入节被使用。包含入口符号的节以及
    所有包含命令行上未定义符号的节将被保留，包含动态对象引用符号的节也会被保留。请注
    意，在构建共享库时，链接器必须假定任何可见符号都被引用。一旦确定了初始的节集合，链
    接器将递归地将其重定位信息引用的任何节标记为已使用。请参阅 --entry、--undefined
    和 --gc-keep-exported。

    此选项可以在进行部分链接（使用选项 -r 启用）时设置。在这种情况下，必须通过
    --entry、--undefined 或 --gc-keep-exported 选项之一，或者通过链接脚本中的 
    ENTRY 命令显式指定要保留的符号的根。

    作为 GNU 扩展，标记有 `SHF_GNU_RETAIN` 标志的 ELF 输入节不会被垃圾回收。

**--print-gc-sections --no-print-gc-sections** ::

    列出所有通过垃圾回收移除的节。列表将打印到标准错误输出。此选项仅在通过
    --gc-sections 选项启用了垃圾回收时才有效。通过在命令行上指定
    --no-print-gc-sections，可以恢复默认行为（即不列出被移除的节）。

**--gc-keep-exported** :::

    当启用 --gc-sections 时，此选项可防止对包含具有默认或受保护可见性的全局符号的未使
    用输入节进行垃圾回收。此选项旨在用于可执行文件，如果不指定该选项，无论包含的符号的
    外部可见性如何，未引用的节都将被垃圾回收。请注意，此选项在链接共享对象时无效，因为
    它是默认开启的。此选项仅支持 ELF 格式的目标。

**--print-memory-usage** ::

    打印使用 MEMORY 链接脚本命令创建的内存区域的已使用大小、总大小和已使用大小。这在嵌
    入式目标上很有用，可以快速查看可用内存的数量。输出格式有一个标题行和每个区域一行。
    它既便于人类阅读，也易于工具解析。以下是输出示例：

    Memory region Used Size Region Size %age Used
    ROM:          256 KB    1 MB       25.00%
    RAM:          32 B      2 GB       0.00%

**--stats** ::

    计算并显示有关链接器操作的统计信息，例如执行时间和内存使用情况。

**--no-whole-archive** ::

    关闭 `--whole-archive` 选项对后续存档文件的影响。

**--whole-archive** ::

    对于 --whole-archive 选项之后在命令行上提到的每个存档文件，将存档中的每个目标文件
    都包含在链接中，而不是在存档中搜索所需的目标文件。这通常用于将存档文件转换为共享
    库，强制将每个目标包含在生成的共享库中。此选项可以使用多次。

    从 gcc 使用此选项时需要注意两点：首先 gcc 不了解此选项，因此必须使用
    -Wl,-whole-archive。其次，在列出存档文件后不要忘记使用
    -Wl,-no-whole-archive，因为 gcc 会将其自己的存档文件列表添加到链接中，而你可能不
    希望此标志影响这些文件。

**--build-id --build-id=style** ::

    请求创建一个 .note.gnu.build-id ELF 注释节或一个 .buildid COFF 节。注释的内容是
    用于唯一标识此链接文件的比特位串。style 可以是 uuid （使用 128 位随机位）、sha1
    （对输出内容的规范部分使用 160 位 SHA1 哈希）、md5（对输出内容的规范部分使用 128 
    位 MD5 哈希），或者 0xhexstring（使用指定为偶数个十六进制数字的选定位串，数字对之
    间的 - 和 : 字符将被忽略）。如果省略 style，则使用 sha1。

    md5 和 sha1 样式生成的标识符在相同的输出文件中始终相同，但在所有不同的输出文件中都
    是唯一的。它并非用于作为文件内容的校验和进行比较。链接文件可能会被其他工具修改，但
    标识原始链接文件的构建 ID 位串不会改变。

    将 style 指定为 none 可禁用命令行上之前任何 --build-id 选项的设置。
