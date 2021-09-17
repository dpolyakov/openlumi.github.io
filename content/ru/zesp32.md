+++

title = "Установка Zesp32  на прошивку с openwrt"

translationKey = "zesp32"
+++


Zesp будет работать так же как и на стоковой прошивке, но ввиду отличия в системных
библиотеках, требуются дополнительные шаги

### 1. Установка nodejs и zesp32

Добавьте репозиторий с пакетами для проекта openlumi

```shell
[ -f /lib/libustream-ssl.so ] && echo "libustream already installed" || opkg install libustream-mbedtls
wget https://openlumi.github.io/openwrt-packages/public.key
opkg-key add public.key
echo 'src/gz openlumi https://openlumi.github.io/openwrt-packages/packages/19.07/arm_cortex-a9_neon' >> /etc/opkg/customfeeds.conf
```

Установите пакеты на шлюз

```shell
opkg update
opkg install node node-npm
wget http://82.146.46.112/fw/ZESPowrt.tar.gz
tar -xzvf ZESPowrt.tar.gz -C /
wget https://raw.githubusercontent.com/openlumi/xiaomi-gateway-openwrt/master/files/zesp32.init -O /etc/init.d/zesp32
chmod +x /etc/init.d/zesp32
wget https://raw.githubusercontent.com/openlumi/xiaomi-gateway-openwrt/master/files/zesp32-package.json -O /opt/app/package.json
/etc/init.d/zesp32 enable
/etc/init.d/zesp32 start
```

### 2. Zigbee прошивка.

Если у вас уже прошита версия для zesp, можно пропустить этот шаг.

Остановите zesp32

```shell
killall node
```

Чтобы прошить на openwrt введите команду

```shell
jnflash /opt/app/util/Zigbee.bin
jntool erase_pdm
```

Это обновит прошивку в модуле jn5169
Теперь можно запустить zesp32 заново

```shell
/etc/init.d/zesp32 restart
```

Интерфейс zesp32 будет доступен на `http://*GATEWAY_IP*:8081/`
