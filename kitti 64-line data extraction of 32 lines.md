kitti 64-line data extraction of 32 lines
 
 

ori link:  https://zhuanlan.zhihu.com/p/71731356


Recently, it is necessary to migrate the kitti data to the target classification of the 32-line radar, so I specially did a job of downsampling the kitti's 64-line data to 32 lines, and only found out that there are pits in the kitti data set.



The first pit is that the point cloud data in the kitti data set should have a big change from the original one. All the information on the Internet says that its horizontal resolution is 0.08 degrees, that is, the number of points in a scan line is about 4500 , but in fact its current data has only about 2000 points per row.



The second pit is because it is disordered data. To extract the data of 32 lines, you need to know which line each point is on. At the beginning, I adopted the idea of finding a turning point to determine the line ownership of each point. The idea is as follows:



The KITTI data set storage is to store the data of the next laser after all the scanning data of one laser is stored, so the identification can be obtained by calculating the difference between the angle values of the point on the horizontal plane.



````python


def kitti_64_to_32(points):
    """
    kitti 64线数据抽取其中偶数线的数据
    :param points:
    :return:
    """
    x_lidar = points[:, 0]  # -71~73
    y_lidar = points[:, 1]  # -21~53

    #  得到水平角度

    angle_diff = np.abs(np.diff(x_radians))

    angle_diff = np.hstack((angle_diff, 0.0001)) # 补一个元素，diff少了一个
    angle_diff_mask = angle_diff > threshold_angle

    #  总共的确有65行。。  重新设置转折点，二个转折点的中点作为新的转折点
    change_point_adjusted = [int((change_point[i]+change_point[i+1])/2) for i in range(len(change_point)-1)]
    # print(change_point_adjusted,len(change_point_adjusted))

    angle_diff_mask = np.zeros(len(angle_diff_mask))

    y_img = np.cumsum(angle_diff_mask)
    # print(np.max(y_img), np.argwhere(angle_diff_mask))

    indices_32 = y_img%2 == 0  # 偶数线点数据
    # indices_32 = y_img==34

    points_32 = points[indices_32,:]
    # points_32 = points[angle_diff_mask,:]

    return points_32



def convert_all():

    save_object_cloud_path = r'D:\KITTI\Object\training\lidar_64_to_32'

    for img_id in range(7481):

        lidar_path = r'D:\KITTI\Object\training\velodyne\%06d.bin' % img_id  ## Path  
        record = np.fromfile(lidar_path, dtype=np.float32).reshape(-1, 4)

        s = time.time()
        points_32s = kitti_64_to_32(record)
        print('time1:', time.time() - s ,record.shape, points_32s.shape)

        np.save(save_object_cloud_path + '\\%06d' % img_id, points_32s)

````



questions to contract me : lonlonago@foxmail.com
