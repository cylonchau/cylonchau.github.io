# jenkins的用户授权与管理


### 需要安装的插件

Role-Based Strategy（可以对构建的项目进行授权管理，让不同的用户管理不同的项目，将测试和生产环境分开）

### 选择授权策略

![image-20221025230035715](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025230035715.png)

当Role-based Authorization Strategy 这个插件安装好之后，授权策略会多出一个Role-Based Strategy 选项，选择此项

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567528458259.png)

### 添加配置权限

系统设置 >> Manage and Assign Roles

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567528458323.png)

### Manage Roles

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567528458295.png)

设置全局角色（全局角色可以对jenkins系统进行设置与项目的操作）
- admin:对整个jenkins都可以进行操作
- root:可以对所有的job进行管理
- other:只有读的权限 other必须有，否则给用户分配角色时分配没有全局role会导致分配失效


### Assign Roles为用户指派角色

项目角色是根据正则匹配的，

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567528458429.png)


![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567528458465.png)


![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567528458386.png)


![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567528458720.png)
