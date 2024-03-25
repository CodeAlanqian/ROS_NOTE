# ROS2笔记

## 编译选项--symlink-install的作用

**注意：ROS2与ROS1不同，修改launch文件后也需要重新编译！为避免这种情况，需要在编译时添加选项 `--symlink-install`选项，这样在修改了launch文件，不用编译也是生效的。**

## 启动文件

ROS2 launch使用python文件编写，需要导入ROS2的包，以便使用ROS2的API对相应的节点进行设置。

### 参数设置

* **launch文件可以传值给下级launch文件**
* **访问launch文件的入口参数**

  有时我们希望能访问到launch文件的入口参数，但是它们通常是LaunchConfiguration类型。常见的形式如下：

`use_sim_time = LaunchConfiguration("use_sim_time")`

这里的use_sim_time是LaunchConfiguration类型的，不能直接用于if判断。如果你直接按如下方式操作：

```python
if use_sim_time
    print(use_sim_time)
```

通常会打印下面的信息，并且if的判句永远是True。

`<launch.substitutions.launch_configuration.LaunchConfiguration object at 0x7f26d46805b0>`

我们可以用OpaqueFunction函数来处理。下面是一个示例：

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, OpaqueFunction
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node, SetParameter


def launch_setup(context, *args, **kwargs):
    use_sim_time = LaunchConfiguration("use_sim_time")
    use_sim_time_str = use_sim_time.perform(context)
    print(f"use_sim_time: {use_sim_time.perform(context)}")
    set_use_sim_time = SetParameter(name="use_sim_time", value=use_sim_time)

    node = Node(
        package="examples_rclcpp_minimal_publisher",
        executable="publisher_lambda",
        name="publisher",
    )

    return [
        set_use_sim_time,
        node,
    ]


def generate_launch_description():
    return LaunchDescription(
        [
            DeclareLaunchArgument("use_sim_time", default_value="false"),
            OpaqueFunction(function=launch_setup),
        ]
    )

```


其中的use_sim_time_str = use_sim_time.perform(context)将参数内容转换为字符串并返回。例如：use_sim_time的值是True的话，use_sim_time_str将是True字符串。

### 构建节点

这个部分主要是写明需要启动哪些节点。并且将节点需要的参数配置好。

```python
 # Nodes launching commands

    start_map_saver_server_cmd = Node(
            package='nav2_map_server',
            executable='map_saver_server',
            output='screen',
            parameters=[configured_params])

    start_lifecycle_manager_cmd = Node(
            package='nav2_lifecycle_manager',
            executable='lifecycle_manager',
            name='lifecycle_manager_slam',
            output='screen',
            parameters=[{'use_sim_time': use_sim_time},
                        {'autostart': autostart},
                        {'node_names': lifecycle_nodes}])
                    
    start_rviz_cmd = Node(
            package='rviz2',
            executable='rviz2',
            name='rviz2',
            arguments=['-d', rviz_config_dir],
            parameters=[{'use_sim_time': use_sim_time}],
            output='screen')
        
    # perform remap so both turtles listen to the same command topic
    forward_turtlesim_commands_to_second_turtlesim_node = Node(
            package='turtlesim',
            executable='mimic',
            name='mimic',
            remappings=[
                ('/input/pose', '/turtlesim1/turtle1/pose'),
                ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel'),
            ]
        )
```

节点中有8个参数。如果不需要配置可以不写，让它保持默认值。

前面已经提到namespace是为了让节点处于不同的命名空间内，这一点可以在上面的图中体现出来。output='screen' 的意思是执行的节点打印的log直接在窗口中输出。除此之外，log也会存在~/.ros/log 目录下（这是默认路径，其实也是可以配置的）。

补充一下其他几个参数

package='nav2_lifecycle_manager' 是指这个节点在哪个ros2包里。executable='lifecycle_manager' 指明需要运行的程序是ros2包中的哪一个执行文件（一个ros包是可以包含多个执行文件的）。

parameters 中可以配置node的参数。这些参数通常会在参数声明部分赋好值，这里直接传给节点即可。同时这里也可以直接传入yaml参数文件。arguments=['-d', rviz_config_dir] 是节点内部实现的参数。通常通过main函数的行参获取。比如下面这个命令：

`run rviz2 rviz2 -d $(ros2 pkg prefix --share turtle_tf2_py)/rviz/turtle_rviz.rviz`

其中-d就是通过main函数传入进去的参数。

下面的参数用于将两个话题名称对等。**左边**是节点内部代码内写明的，**右边**为系统中其他节点提供的话题。

```python
        remappings=[
            ('/input/pose', '/turtlesim1/turtle1/pose'),
            ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel'),
        ]
```


给节点配置参数文件的示例

```python
import os

from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description():
   config = os.path.join(
      get_package_share_directory('launch_tutorial'),
      'config',
      'turtlesim.yaml'
      )

   return LaunchDescription([
      Node(
         package='turtlesim',
         executable='turtlesim_node',
         namespace='turtlesim2',
         name='sim',
         parameters=[config]
      )
   ])
```


### 启动命令

这一部分只需将前面配置好的命令，用add_action方法加入即可。当然，这里其实有两种写法。一种是下面这种。


```python
    # Create the launch description and populate
    ld = LaunchDescription()

    # Set environment variables
    ld.add_action(stdout_linebuf_envvar)

    # Declare the launch options
    ld.add_action(declare_namespace_cmd)
    ld.add_action(declare_use_namespace_cmd)
    ld.add_action(declare_slam_cmd)
    ld.add_action(declare_map_yaml_cmd)
    ld.add_action(declare_use_sim_time_cmd)
    ld.add_action(declare_params_file_cmd)
    ld.add_action(declare_autostart_cmd)

    # Add the actions to launch all of the navigation nodes
    ld.add_action(bringup_cmd_group)

    return ld

```


另一种是直接将启动的内容写在return语句中。

```python
    return LaunchDescription([
        DeclareLaunchArgument(
            'map',
            default_value=map_dir,
            description='Full path to map file to load'),

        DeclareLaunchArgument(
            'params_file',
            default_value=param_dir,
            description='Full path to param file to load'),

        DeclareLaunchArgument(
            'use_sim_time',
            default_value='false',
            description='Use simulation (Gazebo) clock if true'),

        IncludeLaunchDescription(
            PythonLaunchDescriptionSource([nav2_launch_file_dir, '/bringup_launch.py']),
            launch_arguments={
                'map': map_dir,
                'use_sim_time': use_sim_time,
                'params_file': param_dir}.items(),
        ),

        Node(
            package='rviz2',
            executable='rviz2',
            name='rviz2',
            arguments=['-d', rviz_config_dir],
            parameters=[{'use_sim_time': use_sim_time}],
            output='screen'),
    ])
```
