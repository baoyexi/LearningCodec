# sodb rbsp ebsp nalu 总结


#yuv经过编码得到的原始数据就是SODB（String Of Data Bits）。SODB的bit长度不一定8的整数倍，所以需要填充rbsp_trailing_bits( )得到8bit整数倍的rbsp，其转换关系如下

If the SODB is empty (i.e., zero bits in length), the RBSP is also empty.
– Otherwise, the RBSP contains the SODB as follows:
1) The first byte of the RBSP contains the (most significant, left-most) eight bits of the SODB; the next byte of the
RBSP contains the next eight bits of the SODB, etc., until fewer than eight bits of the SODB remain.
2) rbsp_trailing_bits( ) are present after the SODB as follows:
i) The first (most significant, left-most) bits of the final RBSP byte contains the remaining bits of the SODB(if any).
ii) The next bit consists of a single rbsp_stop_one_bit equal to 1.
iii) When the rbsp_stop_one_bit is not the last bit of a byte-aligned byte, one or more rbsp_alignment_zero_bit is present to result in byte alignment.
3) One or more cabac_zero_word 16-bit syntax elements equal to 0x0000 may be present in some RBSPs after the
rbsp_trailing_bits( ) at the end of the RBSP. 

#而EBSP是在RBSP的基础上增加了防止竞争的0x03字节。

#NALU（）是在ebsp的基础上增加了nal_unit_header( )的头。
|Byte stream NAL unit syntax|Descriptor|
 |  :- | :- | 
nal_unit( NumBytesInNalUnit ) { | |	
nal_unit_header( )	| |
NumBytesInRbsp = 0	| |
for( i = 2; i < NumBytesInNalUnit; i++ )| |	
if( i + 2 < NumBytesInNalUnit && next_bits( 24 ) = = 0x000003 ) {| |	
rbsp_byte[ NumBytesInRbsp++ ] 	|b(8) |
rbsp_byte[ NumBytesInRbsp++ ] 	|b(8) |
i += 2	| |
emulation_prevention_three_byte /* equal to 0x03 */ 	|f(8) |
} else	| |
rbsp_byte[ NumBytesInRbsp++ ] 	|b(8) |
}	| |

AVC码流分为Annex-B和AVCC两种格式
HEVC码流分为Annex-B和HVCC两种格式

AVCC格式也叫AVC1，MPEG-4格式，字节对其，因此也叫Byte-Stream Format。用于mp4/flv/mkv/VideoToolbox。
Annex-B格式也叫MPEG-2 transport stream format格式（ts格式），ElementrayStream格式。

结构上区别主要有两点，一个是参数集（pps，sps）的组织格式；一个是分隔
-Annex-B：使用start code分隔NALU；SPS，PPS按流的方式写在头部
-AVCC：使用NALU长度分隔NALU（固定字节，通常为4字节）；在头部包含extradata（或sequence header）的结构体


