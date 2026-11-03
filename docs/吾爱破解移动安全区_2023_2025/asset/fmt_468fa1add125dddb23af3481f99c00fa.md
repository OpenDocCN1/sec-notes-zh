
``` 
    /**
 *   Author: samle
 *  CreateTime: 2024/09/04
 *
 * myfrida_dexdump执行流程
 * 1.内存搜索dex文件特征，保存到result中
 * 2.保存到文件
 * .
 * 
 */

//取map_off偏移后验证该位置是否在dex文件内存区间中，是的话返回，否则返回null
function get_maps_address(dexptr,range_base,range_end){
    var maps_offset = dexptr.add(0x34).readUInt();
    if(maps_offset === 0 ){
        return null;
    }

    var maps_address = dexptr.add(maps_offset);
    if(maps_address  range_end){
        return null;
    }

    return maps_address;

}
//验证了maps_end的两个情况不存在，就是有效地址
function get_maps_end(maps, range_base , range_end){
    var maps_size = maps.readUInt();

    if(maps_size  50){
        return null;
    }

    var maps_end = maps.add(maps_size * 0xc + 4);

    //这里有个疑问，地址是向高处生长的，应该是maps_end > base || maps_end  range_end){

        return null;
    }
    return maps_end;

}
/*
map结构
    map_offset是map_list的文件偏移，通常在dex文件末尾
    map_list有两个成员,map_item有四个成员
    maplist
        uint size
        list map_item[size]
    map_item
        ushort  type
        ushort  unused
        uint    size
        uint    offset
    如果要想知道整个map_list的长度，用size + map_item[size]就可以了
    4 + 12 * size

enum  TYPE_CODES {
    TYPE_HEADER_ITEM = 0x0000,
    TYPE_STRING_ID_ITEM = 0x0001,
    TYPE_TYPE_ID_ITEM = 0x0002,
    TYPE_PROTO_ID_ITEM = 0x0003,
    TYPE_FIELD_ID_ITEM = 0x0004,
    TYPE_METHOD_ID_ITEM = 0x0005,
    TYPE_CLASS_DEF_ITEM = 0x0006,

    TYPE_MAP_LIST = 0x1000,
    TYPE_TYPE_LIST = 0x1001,
    TYPE_ANNOTATION_SET_REF_LIST = 0x1002,
    TYPE_ANNOTATION_SET_ITEM = 0x1003,

    TYPE_CLASS_DATA_ITEM = 0x2000,
    TYPE_CODE_ITEM = 0x2001,
    TYPE_STRING_DATA_ITEM = 0x2002,
    TYPE_DEBUG_INFO_ITEM = 0x2003,
    TYPE_ANNOTATION_ITEM = 0x2004,
    TYPE_ENCODED_ARRAY_ITEM = 0x2005,
    TYPE_ANNOTATIONS_DIRECTORY_ITEM = 0x2006
};
*/
//获取dex头的map_off偏移和dex尾部的偏移是否一致
function verify_by_maps(dexptr,mapsptr){

    var maps_offset = dexptr.add(0x34).readUInt();
    var maps_size = mapsptr.readUInt();

    for(var i = 0; i  range_end){
            return false;
        }

        if(enable_verify_maps){
            var maps_address = get_maps_address(dexptr,range.base,range_end);

            if(!maps_address){
                return false;
            }

            var maps_end = get_maps_end(dexptr,range.base,range_end);
            if(!maps_end){
                return false;
            }

            return verify_by_maps(dexptr,maps_address);
        }
        else{
            return dexptr.add(0x3c).readUInt() == 0x70;
        }
    }

    return false;
}

//通过maps_list的尾部-dex_base求dex的真实长度。可信度极高
function get_dex_real_size(dexptr,range_base,range_end){

    var dex_size = dexptr.add(0x20).readUInt();
    var maps_address = get_maps_address(dexptr,range_base,range_end)
    if(!maps_address){
        return dex_size;
    }
    var maps_end = get_maps_end(maps_address,range_base,range_end)

    if(!maps_end){
        return dex_size;
    }
    return maps_end.sub(dexptr).toInt32();

}
//验证字符串off，类型off，原型off，字段off，方法off的位置是否在文件头后面，dex尾部前面
function verify_ids_off(dexptr,dex_size){

    var string_ids_off = dexptr.add(0x3c).readUInt();
    var type_ids_off   = dexptr.add(0x44).readUInt();
    var proto_ids_off  = dexptr.add(0x4c).readUInt();
    var field_ids_off  = dexptr.add(0x54).readUInt();
    var method_ids_off = dexptr.add(0x5c).readUInt();

    return string_ids_off = 0x70 && type_ids_off = 0x70 &&  proto_ids_off = 0x70 && field_ids_off = 0x70 && method_ids_off = 0x70
}

var pattern = "64 65 78 0a 30 ?? ?? 00";
var DeepPattern = "70 00 00 00";

function search_dex(deepSearch){
       // console.log("deepSearch value is ",deepSearch.toString())
        var result = []

        Process.enumerateRanges("r--").forEach(function(range){
            try{

               // console.log("search_dex begin...")
                //遍历进程的maps文件的可读内存段
                /*
                if(range.file.path.indexOf("testdex") != -1){
                    console.log("rang.base = ",range.base,range.size,range.file.path);
                }
                */

                Memory.scanSync(range.base,range.size,pattern).forEach(function(match){

                    //if(range.file.path.indexOf("testdex") != -1){

                    //}

                    //console.log("rang.base = ",range.base,range.size,range.file.path);
                    //console.log("base",range.base,range.file.path);
                    //排除系统自带dex文件-通过字符串排除
                    if( range.file && range.file.path && 
                        ( range.file.path.startsWith("/data/dalvik-cache/") ||  range.file.path.startsWith("/system/") || range.file.path.startsWith("/apex/com") ) ){
                            return;

                    }
                    /*
                    range.base 输出的是vdex  内存页开始
                    match.address 输出的dex  匹配特征后的开头
                     console.log("range",hexdump(range.base),{
                            offset:0,
                            length:0x70,
                            header:true,
                            ansi:false})
                    console.log("match",hexdump(match.address),{
                        offset:0,
                        length:0x70,
                        header:true,
                        ansi:false})

                    range.base match.address 0x7af96d7000 5586944 0x7af96d7030 8
                    */

                    //console.log("range.base match.address",range.base, range.size,match.address,match.size)
                    //console.log("rang.base = ",range.base,range.size,range.file.path);
                    //验证dex文件的合法性，获取真实长度

                    //console.log("验证dex文件的合法性，获取真实长度，进入前");
                    //var dex_size1 = match.address.add(0x20).readUInt();
                    //console.log("验证dex文件的合法性，获取真实长度 ",dex_size1);
                    if(verify(match.address,range,false)){
                        //console.log("验证dex文件的合法性，获取真实长度，进入后");
                        var dex_size = get_dex_real_size(match.address,range.base,range.base.add(range.size));

                        //数组操作
                        result.push({
                            "addr": match.address,
                            "size": dex_size
                        });
                        //console.log("当前 result length = ",result.length)
                        //console.log("result addr size ",result[0].addr,result[0].size)
                        //减去vdex的头
                        var max_size = range.size - match.address.sub(range.base).toInt32();
                        if(deepSearch && max_size != dex_size){
                            result.push({
                                "addr": match.address,
                                "size": max_size
                            })
                        }
                        //console.log("当前 result length = ",result.length)
                    }

                })

                //如果开启了深度搜索
                if(deepSearch){
                    Memory.scanSync(range.base,range.size,DeepPattern).forEach(function(match){
                        var dex_base = match.address.sub(0x3c);

                        //匹配dex文件头长度0x70，反推dex的基址，如果基址不在这个内存页，则不是一个dex文件
                        if(dex_base  end) {
            return;
        }

        if (!range.protection.startsWith("r")) {
            console.log("Set read permission for memory range: " + base + "-" + range_end);
            Memory.protect(range.base, range.size, "r" + range.protection.substr(1, 2));
        }
    });
}
//没有使用
function memorydump(address, size) {
    var ptr = new NativePointer(address);
    setReadPermission(ptr, size);
    return ptr.readByteArray(size);
}

function fix_header(dex_base, size){

    /*
    dex_size = len(dex_bytes)

    if dex_bytes[:4] != b"dex\n":
        dex_bytes = b"dex\n035\x00" + dex_bytes[8:]

    if dex_size >= 0x24:
        dex_bytes = dex_bytes[:0x20] + struct.Struct("= 0x28:
        dex_bytes = dex_bytes[:0x24] + struct.Struct("= 0x2C and dex_bytes[0x28:0x2C] not in [b'\x78\x56\x34\x12', b'\x12\x34\x56\x78']:
        dex_bytes = dex_bytes[:0x28] + b'\x78\x56\x34\x12' + dex_bytes[0x2C:]

    return dex_bytes

    */

    var magic = [0x64,0x65,0x78,0x0a,0x30,0x33,0x35,0x00]

    if(dex_base.readCString(4) != "dex\n"){
        Memory.writeByteArray(magic,dex_base);
    }

}
function main(){

    var dexsz =[]
    dexsz =  search_dex(false)
    console.error("当前 dexsz length = ",dexsz.length)
    dexsz.forEach(function(value,index,array){

        //dump dex
        //var dex_bytes =  memorydump(value.addr,value.size);
        //不修复文件头了，暂时不清楚内存属性的问题
        // dex_bytes_file =  fix_header(dex_bytes,value.size);
        //fix header
        //写文件到手机
        //写入前确认app拥有读写sd的权限
        var file = new File("/sdcard/Download/samle/"  + value.size + ".dex","wb");
        file.write(Memory.readByteArray(value.addr,value.size))
        file.flush();
        file.close();

        console.log("写入dex 文件大小 ：0x" ,value.size.toString(16));

    //console.log(e.stack)
    })

}
setImmediate(main)

5> 扫描进程内存maps文件，读取所有可读内存段

这一步借助的是Frida提供的进程相关的API
```
