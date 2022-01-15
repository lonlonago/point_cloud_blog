python-parse-livox-lvx-file



The LVX file parsed according to the DJI protocol and displayed with open3d. The LVX file is the data of the horizon lidar downloaded from the official website of DJI. The data of other livox lidars should also be parsed OK.

Put the effect and code directly


The code is very straightforward and simple to write, open the file, parse, display, there are no other steps, you can leave a message if you have any questions.




# 读取雷达数据头部分:

path = r'C:\Users\dataset\data_dust\2021-11-06 17-45-05.lvx'


f = open(path, 'rb')
data = f.read()
total_len = len(data)
f.close()


f = open(path, 'rb')

print("size LvxFilePublicHeader ", struct.calcsize(LvxFilePublicHeader))
data = f.read(struct.calcsize(LvxFilePublicHeader))  #24
print(data)
data_tuple = struct.unpack(LvxFilePublicHeader, data)
print(data_tuple)



data = f.read(struct.calcsize(LvxFilePrivateHeader))  #24+5
print("size LvxFilePrivateHeader ",struct.calcsize(LvxFilePrivateHeader))
data_tuple = struct.unpack(LvxFilePrivateHeader, data)
print(data_tuple)




data = f.read(struct.calcsize(LvxDeviceInfo))  #24+5+59=88
print("size LvxDeviceInfo ",struct.calcsize(LvxDeviceInfo))
data_tuple = struct.unpack(LvxDeviceInfo, data)
print(data_tuple)
print(data_tuple[33])  # device_type is 3 Horizon





#----------------------------------------------------------------------------------
# 读取雷达数据帧部分:


print("size LvxBasePackHeader ",struct.calcsize(LvxBasePackHeader))

print("size LivoxExtendRawPoint ",struct.calcsize(LivoxExtendRawPoint))


print("size LivoxExtendRawPoint0 ",struct.calcsize(LivoxExtendRawPoint0))

LivoxImuPoint = '<' + 'f'*6
print("size LivoxImuPoint ",struct.calcsize(LivoxImuPoint))


FrameHeader = '<' + 'Q' * 3
print("size FrameHeader ",struct.calcsize(FrameHeader))

size_map = [struct.calcsize(LivoxExtendRawPoint0),
            100*9,
            struct.calcsize(LivoxExtendRawPoint),
            96* 10,
            48*28,
            48 *16,
            struct.calcsize(LivoxImuPoint),
            30 *42,
            30 *22
            ]



point_list = []
frame_list = []
idx = 0
print(struct.calcsize(LvxFilePublicHeader) + struct.calcsize(LvxFilePrivateHeader)+ struct.calcsize(LvxDeviceInfo))
current_offset = struct.calcsize(LvxFilePublicHeader) + struct.calcsize(LvxFilePrivateHeader)+ struct.calcsize(LvxDeviceInfo)


vis = o3d.visualization.Visualizer()
vis.create_window(window_name='map', width=1000, height=720)
vis.get_render_option().background_color = np.asarray([0, 0, 0])  # 背景颜色
point_cloud = o3d.geometry.PointCloud()
vis.add_geometry(point_cloud)
to_reset = True


print(total_len, current_offset)
second = 0
while (current_offset < total_len):

    current_frame_offset = 0
    data = f.read(struct.calcsize(FrameHeader))
    data_tuple = struct.unpack(FrameHeader, data)  # 原始格式是元祖，为什么要转array，元祖不能索引
    assert (current_offset == data_tuple[0])
    next_offset = data_tuple[1]
    current_frame_offset += struct.calcsize(FrameHeader)

    while (current_offset + current_frame_offset < next_offset):
        data = f.read(struct.calcsize(LvxBasePackHeader))
        current_frame_offset += struct.calcsize(LvxBasePackHeader)

        data_tuple = struct.unpack(LvxBasePackHeader, data)  # 原始格式是元祖，为什么要转array，元祖不能索引
        data_type = data_tuple[7]
        data_size = size_map[data_type]
        data = f.read(data_size)
        current_frame_offset += data_size

        if data_type == 2:
            idx += 1
            data_tuple = struct.unpack(LivoxExtendRawPoint, data)  # 96*5
            data_unpack = np.array(data_tuple, dtype=int).reshape(-1, 5)  # np.array会多耗时15毫秒
            point_list.append(data_unpack)
        elif data_type == 6:
            # data_tuple = struct.unpack(LivoxImuPoint, data)  #IMU
            pass  # 暂时不需要IMU数据
        elif data_type == 0:
            idx += 1
            # print('here 0')
            data_tuple = struct.unpack(LivoxExtendRawPoint0, data)  # 96*5
            data_unpack = np.array(data_tuple, dtype=int).reshape(-1, 4)  # np.array会多耗时15毫秒
            point_list.append(data_unpack)
        else:
            pass  # 只解析这2种数据，其他的丢弃

        if idx == 1041:  # 每250个数据包为一帧  99936
        # if idx == 2500:  # 每250个数据包为一帧
        # if idx == 250:  # 每250个数据包为一帧
            point_list = np.vstack(point_list)

            point_list = point_list[:, :3] / 1000
            print("point_list", point_list.shape)
            second += 1
            point_cloud.points = o3d.utility.Vector3dVector(point_list)
            # time.sleep(3)

            vis.update_geometry(point_cloud)  # 0.8版本之后才可以这样用
            # vis.update_geometry()
            if to_reset:
                vis.reset_view_point(True)
                to_reset = False

            vis.poll_events()
            vis.update_renderer()
            time.sleep(4)
            point_list = []
            idx = 0

    current_offset += current_frame_offset
    assert (current_offset == next_offset)

print("second ", second)


questions to contract me : lonlonago@foxmail.com
