Rotate the ground truth object detection frame of the image at any angle


Because when detecting an object, the target needs to be rotated at any angle. After the picture is rotated, the frame of the target, that is, the BBOX, will also rotate around the center point accordingly, and there is no good calculation on the Internet. The coordinates of the BBOX after the rotation The code of , I wrote it myself, after rotating 45 degrees, the picture effect is as follows, the picture has been blurred to a certain extent, just pay attention to the green box:




This problem can actually be simplified to calculate the coordinates of a point after rotating around the origin center of the coordinate axis;

The code is as follows, for reference, the previous calculation of dx and dy has some problems, and it will be reposted here after modification:


````python

#coding:utf-8
from PIL import Image
import numpy as np
import os
import math

#这是一个把 BBOX 画在图片上方法，可以自己写一个
#from visualization import draw_bounding_boxes_for_me



def rotate_image_boxes_any_angle(angle, boxes, img_part):
    """
    图片旋转增强,可以实现任意角度旋转，旋转后的图片会自动拓展，图片会变大，
    但是不会补全黑角，相应的box的大小和位置都会自动变化
    :param angle: 旋转角度，0-360度
    :param box:  [int(j) for j in i[2:6]] 四个数值的列表，表示 x1,y1,x2,y2
    :param img_path: 图片路径
    :return:
    """
    angle = angle % 360.0

    # img = Image.open(img_path)
    # ori_img_box = draw_bounding_boxes_for_me(img_part, boxes)

    # img = Image.fromarray(np.uint8(ori_img_box))
    img = Image.fromarray(np.uint8(img_part))

    # h = img.shape[0]
    # w = img.shape[1]
    h = img.size[1]
    w = img.size[0]
    center_x = int(np.floor(w / 2))
    center_y = int(np.floor(h / 2))
    # print('center_x,center_y', center_x, center_y)

    # img_rotate = img.rotate(angle, expand=True)  # 逆时针任意角度度旋转
    img_rotate = make_up_image(img, angle)  # 把黑边框做补充操作的旋转,不需要补充黑边的话去掉这句就行

    angle_pi = math.radians(angle)  # 转化为弧度角度， 逆时针这里到底要不要乘以-1？？？
    # img_rotate.show()

    # 计算某个点绕坐标轴原点中心旋转后的坐标
    def transform(x, y): # angle 必须是弧度
        return (x - center_x) * round(math.cos(angle_pi), 15) + \
               (y - center_y) * round(math.sin(angle_pi), 15) + center_x, \
              -(x - center_x) * round(math.sin(angle_pi), 15) + \
               (y - center_y) * round(math.cos(angle_pi), 15) + center_y


    # 通过四个顶点的坐标变化后的情况得出新的坐标系和原有坐标系的偏移量dx,dy
    # 分为三种情况，w=h,dx,dy 都会一直为正
    # w>h, dx 会出现为负的情况
    # w<h, dy 会出现为负的情况
    xx = []
    yy = []
    for x, y in ((0, 0), (w, 0), (w, h), (0, h)):
        x_, y_ = transform(x, y)
        xx.append(int(x_))
        yy.append(int(y_))
    if min(xx) < 0:
        dx = abs(min(xx))
    else:
        dx = -abs(min(xx))
    if min(yy) < 0:
        dy = abs(min(yy))
    else:
        dy = - abs(min(yy))
    print(angle, angle_pi, 'dx,dy', dx, dy, h, w, xx,yy)


    box_rot = []
    for box in boxes:
        xx = []
        yy = []
        for x, y in ((box[0], box[1]),
                     (box[0], box[3]),
                     (box[2], box[1]),
                     (box[2], box[3])):
            x_, y_ = transform(x, y)
            xx.append(x_)
            yy.append(y_)
        box_rot.append([min(xx) + dx, min(yy) + dy,
                        max(xx) + dx, max(yy) + dy,
                        # box[4]
                       ])
    # print([min(xx), min(yy), max(xx), max(yy)])
    # print("boxrot", box_rot )

    # img_rotate = draw_bounding_boxes_for_me(img_rotate, box_rot,color=19)

    return img_rotate, box_rot


if __name__ == '__main__':
    # 这里自己传入相应的角度，box坐标，图片路径既可
    rotate_image_boxes_any_angle(angle, box, img_path)


````



questions to contract me : lonlonago@foxmail.com
