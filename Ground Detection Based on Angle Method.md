lidar Ground Detection Based on Angle Method


ori link : https://zhuanlan.zhihu.com/p/75659410


The source of the method is: Effificient Online Segmentation for Sparse 3D Laser Scans



Three assumptions for dividing the ground:

1 The lidar and the ground are parallel;

2 The curvature of the ground is small;

3 Some part of the lowest ray must have observed the ground;



The principle of dividing the ground

This method is based on the depth map, which needs to use the depth map to calculate the angle map between the upper and lower lines;

The first non-zero point in the bottom line is used as the starting point for ground detection. If it is less than 30 (or 45) degrees, it is the ground, and then, if the angle transformation between two adjacent lines is less than 5 (or 7) degrees, it is A category that continuously spreads out to label all the angle maps;




How to calculate the angle:






Steps to split:

1 Compression projection _depth_image

2 Repair _depth_image (this step can be removed, the effect is not good)

3 Calculate the angle image angle_image

4 SG smoothing

5 ZeroOutGroundBFS filter ground

Split effect:




The author's original angle threshold, which was used for the previous threshold, did not work very well. Later, after updating it to the threshold written in the paper, the effect became better;



speed:

Python runs relatively slowly and takes 600ms, and the C++ mentioned by the author only needs 6-10ms;



Split code:

````python

def angle_map_obtain(depth_map):
    """
    获取每行的反射点的高度角度差值
    注意：  总的行数会少了一行
    :param depth_map:
    :return:
    """

    yradians_image = depth_map[:, :, 5]  # 64*870*5
    sin_angles = np.sin(yradians_image)
    cos_angles = np.cos(yradians_image)
    zmat = d * sin_angles
    xmat = d * cos_angles
    delta_z = abs(np.diff(zmat, axis=0))
    delta_x = abs(np.diff(xmat, axis=0))
    angle_map = np.arctan2(delta_z, delta_x)  # (63, 870)

    return angle_map




def zero_out_ground_BFS(angle_image, depth_map,
                        # start_threshold=np.radians(30),
                        start_threshold=np.radians(45),
                        # ground_remove_angle=np.radians(7),
                        ground_remove_angle=np.radians(5),
                        dilate=False):
    """
    分割地面的主要代码部分

    """
    depth = depth_map[:, :, 3]
    depth_shape = depth.shape

    # 初始 label , label matrix
    label = 1
    label_matrix = np.zeros(depth_shape)

    # 循环每列
    for j in range(depth_shape[1]):
        while i > 0 and depth[i, j] < 0.001:
            i -= 1  # 它是从最下面往上面找的，也就是从最底下开始找起，

        if angle_image[i, j] > start_threshold:
            continue  # 初始行的角度超过了阈值，肯定不是地面

        if label_matrix[i, j] == 0:
            ground_BFS(i, j, label, label_matrix,
                       depth, angle_image,
                       ground_remove_angle, )

    if dilate:
        kernel = uniform_kernel(9)
        label_matrix = cv2.dilate(label_matrix, kernel, iterations=1)

    return label_matrix


def ground_BFS(i, j, label, label_matrix,
               depth, angle_image,
               ground_remove_angle):
    # BFS 地面打标签部分
    # TODO 用set 更合适，因为会添加进去很多重复的点，可以用set的自动去重；
    while len(Q) > 0:
        Q.pop(0)  # 默认是丢弃最后一个元素， 要放在这里，不然会死循环
        if label_matrix[r, c] > 0:
            continue  # 已经标注
        # label 赋值
        label_matrix[r, c] = label

        # 深度值为0，不纳入计算
        if depth[r, c] < 0.0001:
            continue

        for rn, cn in neighbourhood(r, c):
            # TODO 这里的尺寸还存在着问题
            if rn >= 63 or rn < 0 or cn < 0 or cn >= 869:
                continue  # 超出范围的点，不考虑
            if label_matrix[rn, cn] > 0:
                continue  # 已经标注

            if abs(angle_image[r, c] - angle_image[rn, cn]) < ground_remove_angle:
                Q.append((rn, cn))
````


There is also a part that projects the point cloud to the depth map, and the part that repairs the depth map has not been posted. The projection was written before, and the repaired part has little effect;
