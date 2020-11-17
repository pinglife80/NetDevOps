# SSH CISCO 交换机基本配置

::calendar:2020年11月15日

## 1.通过第三方netmiko模块登录单台设备配置接口

:exclamation:netmiko在安装时，先行查看官方文档对相应依赖包的版本要求，尤其需要注意python的次版本号，否则即使可以安装，但在调用过程中会出现异常。

- 输入：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# @Time    : 2020/11/15 下午11:33
# @Author  : Conan Zieng
# @FileName: ssh_cisco.py
# @Software: PyCharm


from netmiko import ConnectHandler
from pprint import pprint

cisco_iosv = {
    'device_type': 'cisco_ios',
    'host': '192.168.30.250',
    'username': 'dada',
    'password': 'dada'
}

command_set = ['int loo 1', 'ip addr 3.3.1.10 255.255.255.255', 'no shut']
command_show = 'sho ip int br'
with ConnectHandler(**cisco_iosv) as ssh_session:
    # display current system view
    print(f' Firstly, the switch`s current system view is \"{ssh_session.find_prompt()}\"'.strip())
    output = ssh_session.send_config_set(command_set)
    print(f'Secondly, the switch`s current system view is \"{ssh_session.find_prompt()}\"'.strip())
    output += ssh_session.save_config('wr')
    print(f'Finally, the switch`s current system view is \"{ssh_session.find_prompt()}\"'.strip())
    print(output)
    show = ssh_session.send_command_timing(command_show, use_textfsm=True)
    ssh_session.save_config('wr')
    pprint(show)
    # ssh_session current session
    ssh_session.disconnect()#记得关闭session
```

- 输出：

```
Firstly, the switch`s current system view is "CSW#"
Secondly, the switch`s current system view is "CSW#"
Finally, the switch`s current system view is "CSW#"
configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
CSW(config)#int loo 1
CSW(config-if)#ip addr 3.3.1.10 255.255.255.255
CSW(config-if)#no shut
CSW(config-if)#end
CSW#wr
Building configuration...

  Compressed configuration from 3661 bytes to 1795 bytes[OK]
CSW#
[{'intf': 'GigabitEthernet0/1',
  'ipaddr': '10.10.13.1',
  'proto': 'up',
  'status': 'up'},
 {'intf': 'GigabitEthernet0/0',
  'ipaddr': '10.10.12.1',
  'proto': 'up',
  'status': 'up'},
 {'intf': 'GigabitEthernet0/3',
  'ipaddr': 'unassigned',
  'proto': 'up',
  'status': 'up'},
 {'intf': 'GigabitEthernet0/2',
  'ipaddr': '192.168.30.250',
  'proto': 'up',
  'status': 'up'},
 {'intf': 'GigabitEthernet1/0',
  'ipaddr': 'unassigned',
  'proto': 'up',
  'status': 'up'},
 {'intf': 'GigabitEthernet1/1',
  'ipaddr': 'unassigned',
  'proto': 'up',
  'status': 'up'},
 {'intf': 'GigabitEthernet1/2',
  'ipaddr': 'unassigned',
  'proto': 'up',
  'status': 'up'},
 {'intf': 'GigabitEthernet1/3',
  'ipaddr': 'unassigned',
  'proto': 'up',
  'status': 'up'},
 {'intf': 'Loopback0', 'ipaddr': '3.3.3.3', 'proto': 'up', 'status': 'up'},
 {'intf': 'Loopback1', 'ipaddr': '3.3.1.10', 'proto': 'up', 'status': 'up'}]

Process finished with exit code 0
```

- 代码说明

  ①该代码调用第三方模块netmiko 对单台Cisco ios系统设备进行接口配置，其他配置类同；

  ②with...as... 的用法等价于try...finally...  ，在这里时将ConnectHandler这个对象用字典cisco_ios 实例话以后，赋值给变量ssh_session ，在该语句模块中直接调用ssh_session 进行操作，该语法的好处时代码整洁清晰，实例话对象后直接使用，另外在模块中变相使用了异常处理机制，但是在使用with...as... 语句的前提时，要确认 需要调用的对象中必须要定义“\_enter\_” 和“\_exit_”方法 。

  ③其中字符串的strip()方法时去除，字符串首尾的空格，lrstrip()和rstrip()分别时去除左边和右边的空格。

  ④ssh_session实例中的send_config_set 方法时当前的试图直接进入了配置模式，并且可以批量执行命令，该方法中的传递的参数为字符串，或者列表。

  ⑤从3次find_prompt方法的打印结果来看，当前的session 始终保持在登录进来后的试图状态，配置模式是通过上面斯第④条中的方法来进入，但它不影响session当前的模式状态，执行完后依旧在特权模式。session当前的模式取决于登录时给定的用户权限，如果权限默认是用户模式那么要使用send_config_set方法前时就要用enable方法进入到特权模式，还要的实例化对象的参数字典中给’secret‘ 赋值enable 密码。

  ⑥use_textfsm这个参数是将会话中输出的东西经过整理转换成表格的形式，并且数据格式为列表，列表中的数据类型为字典(即每一行的数据)，textfsm数据模板可以自定义。

  ⑦pprint是引入的一个模块，是相同类型的数据格式按行打印出来

  - 补充代码

    下面的代码说明了上面第⑤点，在用户session不是特权模式的情况下如何进入特权模式

  ```python
  #! /usr/lib/python3.7
  # _*_ coding: utf-8 _*_
  
  from netmiko import ConnectHandler
  
  cisco_iosv = {
      'device_type': 'cisco_ios',
      'host': '192.168.30.250',
      'username': 'dada1',
      'password': 'dada1',
      'secret': 'dada1'#enable密码
  }
  command_show = 'sho runn int gi 0/1'
  with ConnectHandler(**cisco_iosv) as connect:
      print(f' Firstly, the switch`s current system view is \"{connect.find_prompt()}\"'.strip())
      output = connect.enable('enable')#进入enable的关键字，之后会自动调用secret变量中的enable密码，进入特权模式
      print(f'Secondly, the switch`s current system view is \"{connect.find_prompt()}\"'.strip())
      output += connect.send_command_timing(command_show,use_textfsm=True)
      connect.disconnect()#记得关闭session
      print(output)
  ```

  ```
  Firstly, the switch`s current system view is "CSW>"
  Secondly, the switch`s current system view is "CSW#"
  enable
  Password: 
  CSW#Building configuration...
  
  Current configuration : 158 bytes
  !
  interface GigabitEthernet0/1
   no switchport
   ip address 10.10.13.1 255.255.255.0
   ip ospf network broadcast
   ip ospf priority 128
   no negotiation auto
  end
  
  
  Process finished with exit code 0
  ```

  ---

  :calendar:2020年11月16日

  ## 2.登录多台设备进行配置

  同样使用netmiko进行多台设备登录

- 输入代码

  ```python
  #!/usr/bin/env python3
  # -*- coding: utf-8 -*-
  # @Time    : 2020/11/16 上午10:16
  # @Author  : Conan Zieng
  # @FileName: ssh_cisco_multi.py
  # @Software: PyCharm
  
  from netmiko import ConnectHandler
  from pprint import pprint
  import logging
  
  
  cisco_iosv = {
      'device_type': 'cisco_ios',
      'host': '192.168.30.250',
      'username': 'dada',
      'password': 'dada',
      'secret': 'dada1',
      'session_log': 'session_log'
      #下面这个参数是开启当前设备的session log,直接会在当前项目目录下生成session_log文件
  }
  hosts_list = ['192.168.30.250', '192.168.30.251', '192.168.30.252']
  command_show = 'sho vlan '
  command_set = ['vlan 100 ', 'name test']
  #这个是调用logging模块生成debug级别的的netmiko模块执行的log，也是在项目目录下生成logging.log的文件
  logging.basicConfig(filename='logging.log', level=5)
  log = logging.getLogger("netmiko")
  
  
  def ssh_cisco_multi_show(hosts_list, command_show):
      '''
      多台设备的登录和单台设备登录的就是将变化的量放在一个list中，通过动态的方式遍历这个list,生成多个ssh_session ，每一次的for循环遍历就是一次单台设备的命令查询操作，当然这个不是高效的方式，后面多台设备会引入其他并行方式
      '''
      
      for host in hosts_list:
          cisco_iosv['host'] = host
          #这个是给不同设备的log后面加上其host地址便于区分
          cisco_iosv['session_log'] = 'session_log_{}'.format(host)
          with ConnectHandler(**cisco_iosv) as ssh_session:
              output = ssh_session.send_command_timing(command_show, use_textfsm=True)
              pprint('Device {} \"{}\" command results is: \n {}'.format(host, command_show, output))
      ssh_session.disconnect()
  
  
  def ssh_cisco_multi_config(hosts_list, command_set):
      for host in hosts_list:
          cisco_iosv['host'] = host
          cisco_iosv['session_log'] = 'session_log_{}'.format(host)
          with ConnectHandler(**cisco_iosv) as ssh_session:
              output = ssh_session.send_config_set(command_set)
              print('Device {} output is: \n {}'.format(host, output))
              ssh_session.disconnect()
  
  
  def main():
      ssh_cisco_multi_show(hosts_list,command_show)
      ssh_cisco_multi_config(hosts_list, command_set)
  
  
  if __name__ == '__main__':
      main()
  
  ```

- 代码输出

  ```
  Device 192.168.30.250 "sho vlan " command results is: 
   [
    {
      "vlan_id": "1",
      "name": "default",
      "status": "active",
      "interfaces": [
        "Gi1/0",
        "Gi1/1",
        "Gi1/2",
        "Gi1/3"
      ]
    },
    {
      "vlan_id": "12",
      "name": "VLAN0012",
      "status": "active",
      "interfaces": []
    },
    {
      "vlan_id": "100",
      "name": "test",
      "status": "active",
      "interfaces": []
    }
   ]
   Device 192.168.30.251 "sho vlan " command results is: 
   [
    {
      "vlan_id": "1",
      "name": "default",
      "status": "active",
      "interfaces": [
        "Gi1/3"
      ]
    },
    {
      "vlan_id": "10",
      "name": "VLAN0010",
      "status": "active",
      "interfaces": []
    },
    {
      "vlan_id": "100",
      "name": "test",
      "status": "active",
      "interfaces": []
    }
   ]
   Device 192.168.30.252 "sho vlan " command results is: 
   [
    {
      "vlan_id": "1",
      "name": "default",
      "status": "active",
      "interfaces": [
        "Gi1/3"
      ]
    },
    {
      "vlan_id": "10",
      "name": "VLAN0010",
      "status": "active",
      "interfaces": []
    },
    {
      "vlan_id": "11",
      "name": "VLAN0011",
      "status": "active",
      "interfaces": []
    },
    {
      "vlan_id": "100",
      "name": "test",
      "status": "active",
      "interfaces": []
    }
   ]
  Device 192.168.30.250 output is: 
   configure terminal
  Enter configuration commands, one per line.  End with CNTL/Z.
  CSW(config)#vlan 100
  CSW(config-vlan)#name test
  CSW(config-vlan)#end
  CSW#
  Device 192.168.30.251 output is: 
   configure terminal
  Enter configuration commands, one per line.  End with CNTL/Z.
  DSW1(config)#vlan 100
  DSW1(config-vlan)#name test
  DSW1(config-vlan)#end
  DSW1#
  Device 192.168.30.252 output is: 
   configure terminal
  Enter configuration commands, one per line.  End with CNTL/Z.
  DSW2(config)#vlan 100
  DSW2(config-vlan)#name test
  DSW2(config-vlan)#end
  DSW2#
  
  Process finished with exit code 0
  ```

- 代码说明

  ①多台设备就是将单台设备的操作迭代若干次，找到不同的迭代变量，这里是IP地址不同，在实际生产环境中迭代基准量无非也就是IP或者hostname去区分设备迭代，其他信息都是相对固定的，如账号/密码这些针对一个管理员通常情况下对设备的管理账号是固定的，另外其他参数也是相对固定的。

  ②在此次的多设备操作中引入了log ，cisco_iosv 字典中的’log_session‘ 其实就是将netmiko自动化执行命令的一系列session log 记录在指定的文件中，而通过logging 模块记录的 不单单是session log ，还可以根据级别去记录netmiko这个模块自身在执行过程中的log。

  :exclamation:上面的log_session参数实际用处还是挺大的，后期可以将这个log存储到DB中用于回溯操作记录。

  ③另外此次的代码实验也重点研究了 对于netmiko 执行生产非格式化数据如何转化为格式化，自动化最关键的就是对执行结果中的响应字段和值进行格式化分离提取，用可操作的数据格式拿到我们响应的信息再进行进一步的处理。其中在netmiko模块下 最关键的就是textFSM 这个数据模块 ，默认情况下networktocode团队已经实现了主流设备的一些常用命令执行结果，ntc-template 就是这个团队已经定义好的模板，我们可以直接使用，在pycharm中安装netmiko时会自动安装，所以在启用use_testfsm 这个参数时，就会调用ntc-template模板中已经定义好的testfsm 模板，前提是我们操作的命令已经定义好了，如果没有那就需要自己研究定义testfsm。另外napalm模块是集成了netmiko和testfsm两者的功能，napalm可以直接完成设备登录的过程，然后执行的结果已经完成了textfsm模板定义的转换过程（前提也是napalm不同的版本支持命令或者厂商不同，这个没法自定义），所以最精髓的还是对textfsm模板的定义掌握，这样我们才可以随心所以获取我们想要的信息。

  下面是通过napalm实现简单的设备登录和show操作，napalm其实就是根深层次的将netmiko和textfsm集成封装起来，对用户来说直接拿来用就成，个性化的东西还是要去研究初始的东西。

  - 补充代码

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# @Time    : 2020/11/17 上午12:14
# @Author  : Conan Zieng
# @FileName: npl_test.py
# @Software: PyCharm
from napalm import get_network_driver
from pprint import pprint
import json

def napala_test():
    driver = get_network_driver('ios')
    device = driver(hostname='192.168.30.250', username='dada', password='dada')
    device.open()
    arp = device.get_arp_table()
    pprint(arp)
    print(json.dumps(arp, indent=2))


def main():
    napala_test()


if __name__ == '__main__':
    main()
```

- 输出结果

```
[{'age': -1.0,
  'interface': 'Vlan12',
  'ip': '10.10.12.1',
  'mac': '50:00:00:01:80:0C'},
 {'age': 70.0,
  'interface': 'Vlan12',
  'ip': '10.10.12.2',
  'mac': '50:00:00:02:80:0C'}]
  [
  {
    "interface": "Vlan12",
    "mac": "50:00:00:01:80:0C",
    "ip": "10.10.12.1",
    "age": -1.0
  },
  {
    "interface": "Vlan12",
    "mac": "50:00:00:02:80:0C",
    "ip": "10.10.12.2",
    "age": 70.0
  }
 ]
 Process finished with exit code 0
```

上面例子的结果，执行的结果数据类型都是list，但是list中的数据类型前者是字典，而后者是json，其实区别在python看了是不大的。所以那个都可以使用。json.dumps(object)是将python对象转换为json格式，json.loads()是将json格式转换为python对象，其中indent参数是数据安装格式缩进显示，打印显示美观。 