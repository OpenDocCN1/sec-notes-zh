
# 分区检测代码片段

```
partitionInfo->StartingOffset.QuadPart = 0;
partitionInfo->PartitionLength.QuadPart = 0;
partitionInfo->HiddenSectors = 0;
}
}
KdPrintEx((DPFLTR_FSTUB_ID, DPFLTR_TRACE_LEVEL, "\n"));
//
// If an error occurred, leave the routine now.
//
if (!NT_SUCCESS( status )) {
    break;
}
if(tableIsValid == FALSE) {
// 
// Invalidate this partition table and stop looking for new ones.
// we'll build the partition list based on the ones we found
// previously.
//
partitionTableCounter--;
break;
}
//
// Now check to see if there are any link entries in this table,
// and if so, set up the sector address of the next partition table.
// There can only be one link entry in each partition table, and it
// will point to the next table.
//
partitionTableEntry = (PPARTITION_DESCRIPTOR) &(((PUSHORT) readBuffer)[PARTITION_TABLE_OFFSET]);
//
// Assume that the link entry is empty.
//
partitionTableOffset.QuadPart = 0;
for (partitionEntry = 1;
     partitionEntry PartitionType)) {
// 
// Obtain the address of the next partition table on the
// disk.  This is the number of hidden sectors added to
// the beginning of the extended partition (in the case of
// logical drives), since all logical drives are relative
// to the extended partition.  The VolumeStartSector will
// be zero if this is the primary parition table.
//
// BUG就在这里，既没有判断同样的扇区是否被访问过，又没有判断新扇区是否是前面的扇区，就卡在这个循环里了，直到最后因为无法分配内存而退出
partitionTableOffset.QuadPart = volumeStartOffset.QuadPart +
    UInt32x32To64(GET_STARTING_SECTOR(partitionTableEntry),
                  SectorSize);
//
// Set the VolumeStartSector to be the begining of the
// second partition (extended partition) because all of
// the offsets to the partition tables of the logical drives
// are relative to this extended partition.
//
if (primaryPartitionTable) {
    volumeStartOffset = partitionTableOffset;
}
//
// Update the maximum sector to be the end of the container
// partition.
//
maxSector = GET_PARTITION_LENGTH(partitionTableEntry);
KdPrintEx((DPFLTR_FSTUB_ID,
           DPFLTR_TRACE_LEVEL,
           "FSTUB: MaxSector now = %#08lx\n",
           maxSector));
//
// There is only ever one link entry per partition table,
// exit the loop once it has been found.
//
break;
}
}
//
// All the other partitions will be logical drives.
//
primaryPartitionTable = FALSE;
} while (partitionTableOffset.HighPart | partitionTableOffset.LowPart);
//
// Detect super-floppy media attempt #1.
// If the media is removable and has an 0xaa55 signature on it and
// is empty then check to see if we can recognize the BPB.  If we recognize
// a jump-byte at the beginning of the media then it's a super floppy.  If
// we don't then it's an unpartitioned disk.
//
if((diskGeometry.MediaType == RemovableMedia) &&
   (partitionTableCounter == 0) &&
   (mbrSignatureFound == TRUE) &&
   (emptyPartitionTable == TRUE)) {
    PBOOT_SECTOR_INFO bootSector = (PBOOT_SECTOR_INFO) readBuffer;
    if((bootSector->JumpByte[0] == 0xeb) ||
       (bootSector->JumpByte[0] == 0xe9)) {
// 
// We've got a superfloppy of some sort.
//
KdPrintEx((DPFLTR_FSTUB_ID,
           DPFLTR_TRACE_LEVEL,
           "FSTUB: Jump byte %#x found "
               "along with empty partition table - disk is a "
               "super floppy and has no valid MBR\n",
           bootSector->JumpByte));
partitionTableCounter = -1;
}
}
//
// If the partition table count is still -1 then we didn't find any
// valid partition records.  In this case we'll build a partition list
// that contiains one partition spanning the entire disk.
//
if(partitionTableCounter == -1) {
    if((mbrSignatureFound == TRUE) ||
       (diskGeometry.MediaType == RemovableMedia)) {
// 
// Either we found a signature but the partition layout was
// invalid (for all disks) or we didn't find a signature but this
// is a removable disk.  Either of these two cases makes a
// superfloppy.
//
KdPrintEx((DPFLTR_FSTUB_ID,
           DPFLTR_TRACE_LEVEL,
           "FSTUB: Drive %#p has no valid MBR. "
               "Make it into a super-floppy\n", DeviceObject));
KdPrintEx((DPFLTR_FSTUB_ID,
           DPFLTR_TRACE_LEVEL,
           "FSTUB: Drive has %#08lx sectors "
               "and is %#016I64x bytes large\n",
           endSector,
           endSector * diskGeometry.BytesPerSector));
if (endSector > 0) {
    partitionInfo = &(*PartitionBuffer)->PartitionEntry[0];
    partitionInfo->RewritePartition = FALSE;
    partitionInfo->RecognizedPartition = TRUE;
    partitionInfo->PartitionType = PARTITION_FAT_16;
    partitionInfo->BootIndicator = FALSE;
    partitionInfo->HiddenSectors = 0;
    partitionInfo->StartingOffset.QuadPart = 0;
    partitionInfo->PartitionLength.QuadPart =
        (endSector * diskGeometry.BytesPerSector);
    (*PartitionBuffer)->Signature = 1;
    partitionNumber = 0;
}
} else {
// 
// We found no partitions.  Make sure the partition count is -1
// so that we setup a zeroed-out partition table below.
//
partitionNumber = -1;
}
}
//
// Fill in the first field in the PartitionBuffer. This field indicates how
// many partition entries there are in the PartitionBuffer.
//
(*PartitionBuffer)->PartitionCount = ++partitionNumber;
if (!partitionNumber) {
// 
// Zero out disk signature.
//
(*PartitionBuffer)->Signature = 0;
}
//
// Deallocate read buffer if it was allocated it.
//
if (readBuffer != NULL) {
    ExFreePool( readBuffer );
}
if (!NT_SUCCESS(status)) {
    ExFreePool(*PartitionBuffer);
    *PartitionBuffer = NULL;
}
#if DBG
if (NT_SUCCESS(status)) {
    FstubDbgPrintDriveLayout(*PartitionBuffer);
}
#endif
return status;
}
```

# 原因分析

原因已经很清楚了，就是`Windows`内核在找分区的时候没有进行更多的判断，导致了死循环，但是这是`Win2k3`的代码，到`Win11`这个`BUG`还没有修复。

# 修复方法

## 解除硬盘逻辑锁

首先需要解除硬盘逻辑锁。

如果是`BIOS`启动中了逻辑锁，可以用`DiskGenius DOS`版（或其他非微软系列的操作系统，如`FreeDOS`，`linux`）删除分区表。

如果是`UEFI`中了逻辑锁，就比较麻烦（有的电脑连`UEFI`设置都进不去），可以考虑把硬盘拆下并接到`linux`电脑上删除分区表。

再进入`PE`，如果不想恢复数据就可以重新分区重装系统了，如果想恢复数据可以找专业恢复数据的地方恢复，下面是手动恢复方法（实验性的，不一定能恢复成功，建议先备份）

## 手动恢复方法

1. 打开`WinHex`，选择工具->打开磁盘->物理驱动器->你的硬盘

2. 选择搜索->查找文本，输入`NTLDR`，编码选择`Unicode`，记下搜索到的扇区编号

3. 如果有此分区的`PBR`备份直接恢复`PBR`备份，这个分区就恢复完了（由于病毒没有破坏`MFTMirr`，所以`MFT`不存在时会自动使用`MFTMirr`里的备份）

如果没有此分区的`PBR`备份，那么就需要恢复`PBR`和`MFT`
