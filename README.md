使用时只需更改如下几部分即可。

# KITTI Odometry to ROSbag

This python tool is aimed to solve the problem that convert kitti odometry to rosbag.Although there is some tool to convert kitti raw data to rosbag. And this tool is developing.we can convert point cloud and pose information to rosbag and pose is convert to tf message.

​     这个python工具用来将下载的Kitti数据集中的Velodyne Points 转为rosbag，并且可以将其中的位姿信息通过tf的形式发布出来。

## run

 

~~~

python wjx_kitti.py -s '00'
~~~
'00' present the point cloud sequences you want to convert.And it exits ground truth information if the sequences between '00' to '11'。

-s 参数代表你要用其中的哪个包  从00-11之间是有groundtruth的.

If you want to change the topic, you may modifiy the code in wjx_kitti.py .


# kitti_odom_to_bag


def main():
    
    parser = argparse.ArgumentParser(description = "Convert KITTI dataset to ROS bag file the easy way!")
    # Accepted argument values
    kitti_types = [ "odom_color" ]
    odometry_sequences = []
    for s in range(22):
        odometry_sequences.append(str(s).zfill(2))
    
    #parser.add_argument("kitti_type", choices = kitti_types, help = "KITTI dataset type")
/////////////***************///////////
    parser.add_argument("dir", nargs = "?", default = '/home/lgw/Documents/bag/odometry/dataset', help = "base directory of the dataset, if no directory passed the deafult is current working directory")
/////////////***************///////////
    parser.add_argument("-s", "--sequence", choices = odometry_sequences,help = "sequence of the odometry dataset (between 00 - 21), option is only for ODOMETRY datasets.")
    args = parser.parse_args()

    bridge = CvBridge()
    compression = rosbag.Compression.NONE
    # compression = rosbag.Compression.BZ2
    # compression = rosbag.Compression.LZ4
    
    # CAMERAS
    cameras = [
        (0, 'camera_gray_left', '/kitti/camera_gray_left'),
        (1, 'camera_gray_right', '/kitti/camera_gray_right'),
        (2, 'camera_color_left', '/kitti/camera_color_left'),
        (3, 'camera_color_right', '/kitti/camera_color_right')
    ]
/////////////***************///////////
    basedir='/home/lgw/Documents/bag/odometry/dataset'
    seq ='00'
/////////////***************///////////
    dataset=pykitti.odometry(basedir,args.sequence )

    second_pose = dataset.poses[1]
    third_velo = dataset.get_velo(2)
    velo_frame_id = 'velo_link'
    velo_topic = '/kitti/velo'
    current_epoch = (datetime.utcnow() - datetime(1970, 1, 1)).total_seconds()

    T_base_link_to_imu = np.eye(4, 4)
    T_base_link_to_imu[0:3, 3] = [-2.71/2.0-0.05, 0.32, 0.93]

    bag= rosbag.Bag("kitti_sequence_{}.bag".format(args.sequence), 'w', compression=compression)
    # dataset.load_timestamps() 
    # dataset.load_poses()
    #save_dynamic_tf(bag, dataset, 'odom',initial_time=1 )
/////////////***************///////////
wjx_save_velo_data2(bag, dataset, velo_frame_id, velo_topic,'00',initial_time=current_epoch)
 /////////////***************///////////
    wjx_save_dynamic_tf(bag, dataset, initial_time=current_epoch)
   

    print("## OVERVIEW ##")
    print(bag)
    bag.close()
            
    
    

if __name__ == '__main__':
    main()
