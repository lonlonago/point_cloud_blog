RANSAC Lidar Ground Detection (2)
 
The last article wrote about two shortcomings of the random sampling consistency method to find the ground:

One is that the wall will be regarded as the ground, and the other is that the slope will be missed, resulting in only a part of the road surface being detected;



This time I will talk about the ideas to solve these two problems:

1 For the first question, we can introduce an assumption, assuming that the radar and the ground are parallel, and this is usually true, then on the basis of this assumption, it can be inferred that the plane we are looking for is perpendicular to the Z axis of the lidar. That is to say, the angle between the normal vector of the plane and the Z axis (0, 0, 1) is very small, then we can filter out the plane of the wall through this assumption;



2 The second problem is mainly because the slope section in the far place is indeed not the same plane as the nearby road, so we can consider segmental fitting, that is, divide the originally fitted road surface, 20 meters (threshold value) Depending on the situation) I use another plane to fit the road outside the road;



3 Another optimization is to speed up the speed of finding planes and avoid finding low-lying planes. I used the height value to filter the random sampling points, because the lidar height of the kitti dataset is 1.73 meters, so use {-1.73 -0.2 ~ -1.73+0.3 } to sample and fit the plane;





code show as below:

````python

def my_ransac_v5(data,
              distance_threshold=0.3,
              P=0.99,
              sample_size=3,
              max_iterations=10000,
              lidar_height=-1.73+0.4,
              lidar_height_down=-1.73-0.2,
              first_line_num=2000,
              alpha_threshold=0.03, # 0.05
              use_all_sample=False,
              y_limit=4,
              ):
    """

    :param data:  N * 4
    :param sample_size:
    :param P :
    :param distance_threshold:
    :param max_iterations:
    :return:
    "
    K = max_iterations # 增加夹角判断后，K初始值不能太小，否则可能一直没有进入最优判断语句


    if not use_all_sample:
        # 不采用第一根线做法，单纯只用高度过滤
        z_filter = data[:,2] < lidar_height  # kitti 高度加0.4
        z_filter_down = data[:,2] > lidar_height_down  # kitti 高度过滤
        filt = np.logical_and(z_filter_down, z_filter)  # 必须同时成立

        first_line_filtered = data[filt,:]
        print('first_line_filtered number.' ,first_line_filtered.shape,data.shape)
    else:
        first_line_filtered = data

    if data.shape[0] < 1900 or first_line_filtered.shape[0] < 180:
        print(' RANSAC point number too small.')
        return None, None, None, None

    L_data = data.shape[0]
    R_L = range(first_line_filtered.shape[0])


    while i < K:
       

        # 计算平面方程系数
        coeffs = estimate_plane(first_line_filtered[s3,:], normalize=False)
        if coeffs is None:
            continue
        # 法向量的模, 如果系数标准化了就不需要除以法向量的模了
        r = np.sqrt(coeffs[0]**2 + coeffs[1]**2 + coeffs[2]**2 )
        # 法向量与Z轴(0,0,1)夹角
        alphaz = math.acos(abs(coeffs[2]) / r)

        # r = math.sqrt(coeffs[0]**2 + coeffs[1]**2 + coeffs[2]**2 )
        # 计算每个点和平面的距离，根据阈值得到距离平面较近的点数量
        # d = np.abs(np.matmul(coeffs[:3], data.T) + coeffs[3]) / r
        d = np.divide(np.abs(np.matmul(coeffs[:3], data[:,:3].T) + coeffs[3]), r)
        d_filt = np.array(d < distance_threshold)
        d_filt_object = ~d_filt

        near_point_num = np.sum(d_filt,axis=0)

        # 为了避免将平直墙面检测为地面，必须将夹角加入判断条件，
        # 如果只用内点数量就会导致某个
        if near_point_num > max_point_num and alphaz < alpha_threshold:
            max_point_num = near_point_num

            best_model = coeffs
            best_filt = d_filt
            best_filt_object = d_filt_object

            # # 与Z轴(0,0,1)夹角
            # alpha = math.acos(abs(coeffs[2]) / r)
            alpha = alphaz

            w = near_point_num / L_data

           
       
            # sd_w = np.sqrt(p_no_outliers) / wn
            K = (math.log(1-P) / math.log(p_no_outliers)) #+ sd_w

        i += 1


        if i > max_iterations:
            print(' RANSAC reached the maximum number of trials.')
            return None,None,None,None

    print('took iterations:', i+1, 'best model:', best_model,
          'explains:', max_point_num)
    # 返回地面坐标，障碍物坐标，模型系数，夹角
    return np.argwhere(best_filt).flatten(),np.argwhere(best_filt_object).flatten(), best_model, alpha



# @profile
def my_ransac_segment(data,segment_x=20):
    """
    分段拟合地面

    :param data:  N * 4
    :param sample_size:
    :param P :
    :param distance_threshold:
    :param max_iterations:
    :return:
    """
    data0_20 = data[data[:,0] <= segment_x, :]
    data20plus = data[data[:,0] > segment_x, :]

    indices0_20, indices20, model20, alpha_z0 = my_ransac_v5(data0_20,
                                                           lidar_height=-1.73+0.2,
                                                           )

    indices20plus, indices2, model2, alpha_z = my_ransac_v5(data20plus)



    if (indices20plus is not None) and (indices0_20 is not None):
        return np.vstack((
            data0_20,
            data20plus)), np.hstack(( indices0_20, indices20plus+data0_20.shape[0]))
    elif (indices20plus is not None) and (indices0_20 is None):
        return np.vstack((
            data20plus,
            data0_20)), indices20plus
    elif (indices20plus is None) and (indices0_20 is not None):
        return np.vstack((
            data0_20,
            data20plus)), indices0_20
    else:
        return None, None

````


The same as last time, 100 pieces of data were tested, and the performance was very good. The wall problem was solved, and basically no large road surface would be missed;



The threshold of the angle between the normal vector and the Z axis is 0.05 radians, which is about 2-3 angles, and 0.03 does not seem to be much different;



The distance threshold of the segment is 20, which means that the road after the distance of 20 meters is fitted by a plane, and the front is fitted by another plane. I only focus on the road with x>0, and the radar data behind the car is directly filtered out. , so if you are using all the data,

It may also be a little bit not well fitted.



height filter threshold {-1.73-0.2 ~ -1.73+0.3 };

Due to the two-piece fitting, the overall time is almost twice as long as before, about 38ms;





Overall, the effect has been very good, and finally let me talk about its advantages and disadvantages in my opinion:

advantage:

1 is relatively fast;

2 Compared with the height difference and other grid height difference methods, it will not falsely detect the plane that is parallel to the ground but actually in the air;



shortcoming:

1 Since the plane finding is randomly sampled, it may take a long time in one frame and a short time in another frame, but at present, there has not been a very exaggerated situation, basically dozens of found several times or hundreds of times;

2 Inability to do anything about the sloped road beside the road;


questions to contract me : lonlonago@foxmail.com
