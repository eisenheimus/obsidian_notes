**netstat**  - список приложений слушающие порты
**dsa.msc** - AD
**gpmc.msc** - GPO
**compmgmt**.msc - управление ПК
**hostname** - имя ПК
**ipconfig** - ip и маска сети
**nslookup** - получить адрес DNS-сервера в домене.
**nslookup [mysite.ru]** - получить ip сервера по доменному имени
**whoami** - имя пользователя
**ncpa.cpl** - управление сетью

  

**net use \\localhost /user:liwest\gorchakova Qwe1234!** - проверить логин пароль и доступность учетки


**Win**

Проверить открыт ли порт - Test-NetConnection -ComputerName [mx3.liwest.ru](http://mx3.liwest.ru) -Port 465

netstat  - список приложений слушающие порты

dsa.msc - AD

gpmc.msc - GPO

compmgmt.msc - управление ПК

hostname - имя ПК

ipconfig - ip и маска сети

nslookup - получить адрес DNS-сервера в домене.  
nslookup [mysite.ru] - получить ip сервера по доменному имени

whoami - имя пользователя

net use \\localhost /user:liwest\gorchakova Qwe1234! - проверить логин пароль и доступность учетки


Конфигурация ПК:  
wmic cpu get name - получить модель процессора  
wmic memorychip get Capacity  
diskpart -> list disk - получить объем дисков SSD/HDD  
wmic diskdrive get model, size

|   |   |
|---|---|
|Главная|ms-settings:|
|Диспетчер авторизации|azman.msc|
|Сертификаты - Локальный компьютер|certlm.msc|
|Сертификаты - Текущий пользователь|certmgr.msc|
|Службы компонентов|comexp.msc|
|Управление компьютером|compmgmt.msc|
|Диспетчер устройств|devmgmt.msc|
|Групповые политики - Конфигурация пользователя|DevModeRunAsUserConfig.msc|
|Управление дисками|diskmgmt.msc|
|Просмотр событий|eventvwr.msc|
|Общие папки|fsmgmt.msc|
|Редактор групповых политик|gpedit.msc|
|Локальные пользователи и группы|lusrmgr.msc|
|Монитор производительности|perfmon.msc|
|Управление печатью|printmanagement.msc|
|Результирующая политика|rsop.msc|
|Локальная политика безопасности|secpol.msc|
|Службы|services.msc|
|Планировщик заданий|taskschd.msc|
|Управление доверенным платформенным модулем (TPM)|tpm.msc|
|Брандмауэр|WF.msc|
|Элемент управления WMI (локальный)|WmiMgmt.msc|