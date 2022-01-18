wireshark recording UDP data packet fragmentation reassembly



For UDP, if it is found that the data is too large, the IP layer will automatically cut and fragment the data, but usually we will not find any impact at the application layer, because the fragmented data has been automatically merged, but if you use wireshark The recorded data will be fragmented, but there will be no reorganization. At this time, we need to manually reorganize the data, similar to the following situation:




It can be seen that a data is divided into 8 data shards, but their IDs are the same, and then have different offsets, then we can reorganize the shards of each group according to these two characteristics.



Just use the dpkt tool, the code is as follows.


````python

#coding:utf-8

import dpkt


def main(file_path):

    # f = open(file_path) # This is written under python2,
    f = open(file_path, mode='rb') #python3
    try:
        pcap = dpkt.pcap.Reader(f) # First parse in .pcap format, if not, parse in pcapng format
    except:
        print("it is not pcap... format, pcapng format...")
        pcap = dpkt.pcapng.Reader(f)
        # Next, you can further analyze the pcap. Remember that it is best to use f.close() to close the open file after the end of use, although after the program is finished,
        # The system will turn off by itself, but it is essential to develop good habits. In the current variable pcap, each single packet is stored in the format of "time stamp: single packet"

    first_flag = 1
    raw_data = []
    # Separate the timestamp from the packet data, and parse it layer by layer, where ts is the timestamp, and buf stores the corresponding packet
    for (ts, buf) in pcap:
        try:
            eth = dpkt.ethernet.Ethernet(buf) # unpack, physical layer
            if not isinstance(eth.data, dpkt.ip.IP): # Unpack, network layer, determine whether the network layer exists,
                continue
            ip = eth.data
            print(ip.id, ip.offset)

            # if not isinstance(ip.data, dpkt.tcp.TCP): # Unpack to determine whether the transport layer protocol is TCP, that is, when you only need TCP, it can be used to filter
            # continue
            # if not isinstance(ip.data, dpkt.udp.UDP):#Unpack, determine whether the transport layer protocol is UDP; the fragmented node is ip.data.data, otherwise it is ip.data
            # continue

            if first_flag:
                raw_data.append(ip.data.data)
            else:
                # if (id_flag != ip.id and ip.offset == 0):
                if (ip.offset == 0):
                    # You can make a stricter limit on the number of UDP fragments each time, because basically the number of fragments of the same data source is fixed, for example, each time 3000 bytes are sent, then each time is divided into 2 fragments
                    # For example, each data here is divided into 9 fragments, then I only need the data of 9 fragments, and the data of not 9 fragments will be discarded
                    # if len(raw_data) != 9:
                    # raw_data.clear()
                    # raw_data.append(ip.data.data)
                    # continue
                    result = b""
                    for i in raw_data:
                        result += i
                    tempeth.data.data = result
                    writer.writepkt(tempeth, ts=ts) # If the ts parameter is not added, the timestamp of this packet defaults to the current time!
                    newpacpfile.flush()
                    raw_data.clear()
                    if not isinstance(ip.data.data, bytes): # There may be other types of data
                        continue
                    raw_data.append(ip.data.data)
                else:
                    # print(len(raw_data), "len(raw_data)", type(ip.data))
                    if not isinstance(ip.data, bytes): # There may be other types of data
                        continue
                    raw_data.append(ip.data)
                    tempeth = eth

            first_flag = 0

        except Exception as err:
            print( "[error] %s" % err)
    f.close()






if __name__ == '__main__':
    path = "0325_1024.pcap"
    newpacpfile = open("new.pcap", "wb")
    writer = dpkt.pcap.Writer(newpcapfile)
    main(path)
    newpacpfile.close()

````

The final effect is as follows:
https://zhuanlan.zhihu.com/p/360163299


As can be seen from the data capacity, the data of multiple shards has been combined, but there are still shortcomings. The tag of each shard is still the tag of the shard.
