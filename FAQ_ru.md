# Вопросы и ответы по загрузке linux на android tv box

#### Какие инструменты понадобятся на windows?

Нужно установить wsl и установить в него:

* `sudo apt-get install u-boot-tools` - для сборки бинарных скриптов `aml_autoscript` и для конвертирования linux kernel форматов

#### Как происходит дефолтная загрузка android на tv box от вендора на Amlogic s[89]xx?

В подавляющем большинстве там прописано что-то вроде:

```
factory_reset_poweroff_protect=echo wipe_data=${wipe_data}; echo wipe_cache=${wipe_cache};if test ${wipe_data} = failed; then run init_display; run storeargs;if mmcinfo; then run recovery_from_sdcard;fi;if usb start 0; then run recovery_from_udisk;fi;run recovery_from_flash;fi; if test ${wipe_cache} = failed; then run init_display; run storeargs;if mmcinfo; then run recovery_from_sdcard;fi;if usb start 0; then run recovery_from_udisk;fi;run recovery_from_flash;fi;
update=run usb_burning; run sdc_burning; if mmcinfo; then run recovery_from_sdcard;fi;if usb start 0; then run recovery_from_udisk;fi;run recovery_from_flash;
```

```
recovery_from_sdcard=setenv bootargs ${bootargs} aml_dt=${aml_dt} recovery_part={recovery_part} recovery_offset={recovery_offset};if fatload mmc 0 ${loadaddr} aml_autoscript; then autoscr ${loadaddr}; fi;if fatload mmc 0 ${loadaddr} recovery.img; then if fatload mmc 0 ${dtb_mem_addr} dtb.img; then echo sd dtb.img loaded; fi;wipeisb; bootm ${loadaddr};fi;
recovery_from_udisk=setenv bootargs ${bootargs} aml_dt=${aml_dt} recovery_part={recovery_part} recovery_offset={recovery_offset};if fatload usb 0 ${loadaddr} aml_autoscript; then autoscr ${loadaddr}; fi;if fatload usb 0 ${loadaddr} recovery.img; then if fatload usb 0 ${dtb_mem_addr} dtb.img; then echo udisk dtb.img loaded; fi;wipeisb; bootm ${loadaddr};fi;
```

Т.е. при запуске recovery mode (чаще всего загрузка с зажатой кнопкой питания или кнопкой внутри AV выхода), происходит попытка найти  aml_autoscript на единственном fat разделе mmc или usb и запустить его.

#### Как запустить загрузку с разных разделов на mmc, как это делает Armbian?

Нужно добавить в переменные окружения u-boot от вендора шаг поиска и загрузки на разных разделах.

```
setenv bootcmd 'echo "Start autoscript..."; echo; echo; run start_autoscript; echo "Start storeboot..."; echo; echo; run storeboot;'
setenv start_autoscript 'echo "Try aml_autoscript..."; mmcinfo && run start_mmc_autoscript; usb start && run start_usb_autoscript; run start_emmc_autoscript'
setenv start_mmc_autoscript 'echo "Try start mmc autoscript..."; echo; echo;run start_mmc_0_autoscript; run start_mmc_01_autoscript; run start_mmc_11_autoscript;'
setenv start_mmc_0_autoscript 'echo "Try start mmc 0..."; echo; echo;if fatload mmc 0 ${loadaddr} aml_autoscript; then autoscr ${loadaddr}; fi;'
setenv start_mmc_01_autoscript 'echo "Try start mmc 0:1..."; echo; echo;if fatload mmc 0:1 ${loadaddr} aml_autoscript; then autoscr ${loadaddr}; fi;'
setenv start_mmc_11_autoscript 'echo "Try start mmc 1:1..."; echo; echo;if fatload mmc 1:1 ${loadaddr} aml_autoscript; then autoscr ${loadaddr}; fi;'
setenv start_usb_autoscript 'fecho "Try start usb..."; echo; echo;or usbdev in 0 1 2 3; do fatload usb ${usbdev} ${loadaddr} aml_autoscript && autoscr ${loadaddr}; done'
setenv start_emmc_autoscript 'echo "Try start emmc_autoscript mmc 1..."; echo; echo;if fatload mmc 1 ${loadaddr} emmc_autoscript; then autoscr ${loadaddr}; fi;'
```

И если нужно сохранить это действие в постоянной памяти, то можно это сделать написав `saveenv`.

Даные переменные можно прописать напрямую через UART подключение в консоль u-boot, либо через сборку `aml_autoscript`, который можно загрузить из recovery режима или при помощи стандартной утилиты обновления прошивки в android.

Полный скрипт можно найти тут [https://github.com/EvgeniyBlinov/x96-uboot/blob/main/script/s905_init_aml_autoscript.txt](https://github.com/EvgeniyBlinov/x96-uboot/blob/main/script/s905_init_aml_autoscript.txt)

Инструкция по сборки [https://github.com/EvgeniyBlinov/x96-uboot/blob/main/README.md](https://github.com/EvgeniyBlinov/x96-uboot/blob/main/README.md)

