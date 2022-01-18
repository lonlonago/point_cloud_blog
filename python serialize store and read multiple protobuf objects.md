python serialize store and read multiple protobuf objects
 
First define the problem to be solved: Since protobuf stores and transmits data very fast, we hope to use it to store and read data. There are multiple protobuf objects in the stored data, but when reading, only read To the last one, for example: I sequentially store 10 protobuf objects to the binary file, but when reading, only the last one can be read. This article proposes a solution to this problem.



[Protobuf] Generation and parsing of proto binary files (with complete source code) - Programmer Sought



The article linked above is also trying to solve this problem, but the idea is slightly different, you can also refer to it, its idea is:

Before each piece of serialized binary data, a 4-byte content is placed, and this content is used to save the byte length of the next binary data.
The byte length is obtained by:
proto_len = obj.ByteSize()


My idea is to write another TXT file, which is specially used to save the byte length of each protobuf object. This solution is not as elegant as the above blog, but the number and difficulty of the implemented code are much reduced. Take python code as an example, show it process.



````python

path = r'point.txt'
pathlen = r'point_len.txt'


import ImuPointCloud_pb2


#-------------------------save part---------------------------------------------

b = ImuPointCloud_pb2.Cloud()  # 第一个对象
b.id = 11
po = b.Points.add()
po.x = 11
po.y = 11
po.z = 11
b.latitude = 11.88
b.lon = 11
b.yaw = 11
b.speed = 11.4
b.yawRate = 11.99
b.imuTime = 11.99
b.lidarTime = 11

b1 = ImuPointCloud_pb2.Cloud()  # 第二个对象
b1.id = 22
po1 = b1.Points.add()
po1.x = 22
po1.y = 22
po1.z = 22
b1.latitude = 22.88
b1.lon = 22
b1.yaw = 22
b1.speed = 22.4
b1.yawRate = 22.99
b1.imuTime = 22.99
b1.lidarTime = 22

flen = open(pathlen, 'w')
with open(path, "wb") as f:
    bb = b.SerializeToString()
    bb1 = b1.SerializeToString()
    # print(len(bb), len(bb1))

    flen.write(str(len(bb)))   # 每次写一个对象的同时也写入字节长度
    flen.write('\n')
    f.write(b.SerializeToString())

    flen.write(str(len(bb1)))
    flen.write('\n')
    f.write(b1.SerializeToString())

flen.close()



#-------------------------read part---------------------------------------------

flen = open(pathlen, 'r')
lens = flen.readlines()

a = ImuPointCloud_pb2.Cloud()
try:
    with open(path, 'rb') as f:
        for le in lens:
            print(int(le))
            a.ParseFromString(f.read(int(le)))
            print(a)
except IOError:
    print(path+ ": not found")

````



The reasons for this phenomenon are as follows:

if you want to write multiple messages to a single file or stream, it is up to you to keep track of where one message ends and the next begins. The Protocol Buffer wire format is not self-delimiting, so protocol buffer parsers cannot determine where a message ends on their own. The easiest way to solve this problem is to write the size of each message before you write the message itself. When you read the messages back in, you read the size, then read the bytes into a separate buffer , then parse from that buffer. (If you want to avoid copying bytes to a separate buffer, check out the CodedInputStream class (in both C++ and Java) which can be told to limit reads to a certain number of bytes.)

If you want to write multiple messages to a single file or stream, it's up to you to keep track of where one message ends and the next starts. The Protocol Buffer wire format is not self-delimited, so the Protocol Buffer parser cannot determine on its own where the message ends. The easiest way to fix this is to write down the size of each message before writing the message itself. When you read the message back, you read the size, then read the bytes into a separate buffer, and parse from that buffer. (If you want to avoid copying bytes to a separate buffer, check out the CodedInputStream class (in C++ and Java), which can be told to limit reads to a certain number of bytes.)

From the above description, it can be said that the format of PB is non-descriptive. ParFromIstream will read the entire stream, parse the key-value pair in it, and determine the value of the key (the key is identified by the field number) to set the value.


Simply put, the multiple messages stored in protobuf are not self-descriptive, and you must determine the start and end positions of multiple messages yourself.



Another point to note is that when parsing with C++, if the number of bytes of the message is very large, you cannot use a temporary variable to store the message at this time, you must use a global variable.


````python
for(const int le: len_vec){
result_proto.Clear();
char* temp = &gReadBuf[0]; // gReadBuf is a large array whose capacity matches the size of the message you want to parse
ifsp.read(temp, le);
string temp2(temp, le);
result_proto.ParseFromString(temp2);

}
````

email: lonlonago@foxmail.com



