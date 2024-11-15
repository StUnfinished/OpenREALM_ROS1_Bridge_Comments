# 从ROS的参数服务器中读取相关配置信息
以`catkin_ws/src/OpenREALM_ROS1_Bridge/realm_ros/launch/alexa_noreco.launch`这个文件为例进行讲解，其余launch文件的参数信息配置写法基本一样的，可以依此类推。
alexa_noreco.launch的部分代码如下:

```
<arg name="camera_id" default="alexa"/>
<arg name="topic_adapter" default="/realm/$(arg camera_id)/input"/>
<arg name="topic_pose_est" default="/realm/$(arg camera_id)/pose_estimation/frame"/>
<arg name="topic_dense" default="/realm/$(arg camera_id)/densification/frame"/>
<arg name="topic_surf" default="/realm/$(arg camera_id)/surface_generation/frame"/>
<arg name="topic_rect" default="/realm/$(arg camera_id)/ortho_rectification/frame"/>
<arg name="topic_mosaic" default="/realm/$(arg camera_id)/mosaicing/frame"/>

<!-- pkg表示这个节点在realm_ros功能包下运行，type表示realm_ros功能包中的可执行文件realm_exiv2_grabber -->
<!-- name表示这个节点名是realm_exiv2_grabber，output = “screen”表示将结果输出到终端上 -->
<!-- param标签用于在参数服务器中定义一些参数，话题中带的信息 -->
<node pkg="realm_ros" type="realm_exiv2_grabber" name="realm_exiv2_grabber" output="screen">
    <param name="config/id" type="string" value="$(arg camera_id)"/>
    <param name="config/input" type="string" value="/home/yill/download/test_img_1/"/>
    <param name="config/rate" type="double" value="10.0"/>
    <param name="config/profile" type="string" value="alexa_noreco"/>
</node>
```

里面的`<param>`参数在ROS节点启动后会上传到参数服务器中，用户在cpp文件中可以通过创建ROS节点句柄`ros::NodeHandle`，然后以`.param()`方法获取相应的参数，然后将其存到变量中。<br>
比如要从参数服务器中读取到`<param name="config/id" ...>`这个参数信息，需要先在cpp文件中创建ROS节点句柄`ros::NodeHandle param_nh("~");`，然后将该参数存到变量`string _id_node`中。上述要求就可以通过这句语句实现：`param_nh.param("config/id", _id_node, std::string("uninitialised"));` <br>
在catkin_ws/src/OpenREALM_ROS1_Bridge/realm_ros/src/realm_ros_lib/grabber_exiv2_node.cpp里面，通过`void Exiv2GrabberNode::readParams()`函数实现读取ROS参数服务器内相应参数。<br>
此处再补充解释一下ROS节点句柄`ros::NodeHandle param_nh("~");`。对于当前节点的私有命名空间，通过使用`~`，所有后续的参数读取和话题发布都将在这个命名空间下进行，这样可以避免与其他节点的命名冲突。
例如，假设当前节点的名称是`my_node`，则命名空间"~"相当于`/my_node`。