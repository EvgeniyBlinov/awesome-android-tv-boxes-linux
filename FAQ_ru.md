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
