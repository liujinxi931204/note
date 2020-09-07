在Redis中，根目录中有一个Redis配置文件(redis.conf)，可以通过Redis CONFIG命令获取和设置所有的redis配置  
句法：  
`CONFIG GET CONFIG_SETTING_NAME`  
例如：  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/09/07/1599464279736-1599464279819.png)  
要获取所有的配置设置，请使用"*"代替"CONFIG_SETTING_NAME"  
即`CONFIG GET *`  
编辑配置  
1. 可以直接编辑redis.conf
2. 通过`CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE`的命令设置  
例如：