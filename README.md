# 🔧 KmdfHelloWorld — Windows Kernel Mode Driver

> Первый драйвер режима ядра на Windows, написанный с нуля с использованием KMDF и отлаженный через WinDbg.

---

## 📋 О проекте

Это учебный проект по разработке **драйвера режима ядра (Kernel Mode Driver)** для Windows с использованием **Windows Driver Framework (KMDF)**. Драйвер выводит отладочное сообщение при загрузке и успешно запускается на виртуальной машине под управлением Windows 10.

---

## 🛠️ Стек технологий

| Инструмент | Версия |
|---|---|
| Visual Studio | 2022 (v17.13) |
| Windows Driver Kit (WDK) | 10.0.26100.1 |
| Windows SDK | 10.0.26100.0 |
| Target OS | Windows 10 x64 |
| Виртуальная машина | Oracle VirtualBox |
| Отладчик | WinDbg 10.0.28000 |

---

## 📁 Структура проекта

```
KmdfHelloWorld2/
├── Driver.c          # Исходный код драйвера
├── KmdfHelloWorld2.inf   # Файл установки драйвера
└── x64/Debug/
    ├── KmdfHelloWorld2.sys   # Скомпилированный драйвер
    ├── KmdfHelloWorld2.inf   # INF файл
    └── kmdfhelloworld2.cat   # Каталог безопасности
```

---

## 💻 Код драйвера

```c
#include <ntddk.h>
#include <wdf.h>

DRIVER_INITIALIZE DriverEntry;
EVT_WDF_DRIVER_DEVICE_ADD KmdfHelloWorldEvtDeviceAdd;

NTSTATUS DriverEntry(
    _In_ PDRIVER_OBJECT  DriverObject,
    _In_ PUNICODE_STRING RegistryPath
)
{
    NTSTATUS status = STATUS_SUCCESS;
    WDF_DRIVER_CONFIG config;

    KdPrintEx((DPFLTR_IHVDRIVER_ID, DPFLTR_INFO_LEVEL,
               "KmdfHelloWorld: DriverEntry\n"));

    WDF_DRIVER_CONFIG_INIT(&config, KmdfHelloWorldEvtDeviceAdd);

    status = WdfDriverCreate(DriverObject, RegistryPath,
                             WDF_NO_OBJECT_ATTRIBUTES,
                             &config, WDF_NO_HANDLE);
    return status;
}

NTSTATUS KmdfHelloWorldEvtDeviceAdd(
    _In_    WDFDRIVER       Driver,
    _Inout_ PWDFDEVICE_INIT DeviceInit
)
{
    UNREFERENCED_PARAMETER(Driver);
    NTSTATUS status;
    WDFDEVICE hDevice;

    KdPrintEx((DPFLTR_IHVDRIVER_ID, DPFLTR_INFO_LEVEL,
               "KmdfHelloWorld: KmdfHelloWorldEvtDeviceAdd\n"));

    status = WdfDeviceCreate(&DeviceInit,
                             WDF_NO_OBJECT_ATTRIBUTES, &hDevice);
    return status;
}
```

---

## 🚀 Шаги выполнения

### 1. Установка инструментов
- Visual Studio 2022 с компонентами C++ и Spectre-библиотеками
- Windows SDK 10.0.26100.0
- Windows Driver Kit (WDK) 10.0.26100.1
- Расширение WDK для Visual Studio

### 2. Создание проекта
- Шаблон: **Kernel Mode Driver, Empty (KMDF)**
- Конфигурация сборки: **Debug x64**
- SDK версия: **10.0.26100.0**

### 3. Сборка драйвера
Успешная сборка генерирует три файла: `.sys`, `.inf`, `.cat`

### 4. Настройка виртуальной машины
```cmd
# Отключение проверки цифровых подписей
bcdedit.exe -set loadoptions DISABLE_INTEGRITY_CHECKS
bcdedit.exe -set TESTSIGNING ON

# Настройка отладки через COM-порт
bcdedit /debug on
bcdedit /dbgsettings serial debugport:1 baudrate:115200
```

### 5. Установка драйвера как сервиса
```cmd
sc create KmdfHelloWorld2 type= kernel start= auto error= normal binpath= c:\1\KmdfHelloWorld2.sys
```

### 6. Настройка фильтра отладочной печати
В реестре:
```
HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Debug Print Filter
└── IHVDriver = DWORD: 0xFFFFFFFF
```

### 7. Подключение WinDbg
```
File → Kernel Debug → COM
Port: \\.\pipe\com_1
Baudrate: 115200
[x] Pipe
```

---

## ✅ Результат

После запуска сервиса в WinDbg появляется сообщение:

```
KmdfHelloWorld: DriverEntry
```

Это подтверждает успешную загрузку драйвера в ядро Windows!

---

## 📚 Что я узнал

- Архитектура драйверов режима ядра Windows (KMDF)
- Точка входа `DriverEntry` и её роль в инициализации драйвера
- Callback `EvtDeviceAdd` для обработки обнаружения устройств
- Настройка тестового режима Windows для загрузки неподписанных драйверов
- Ядерная отладка через последовательный порт с WinDbg
- Работа с виртуальными машинами для безопасного тестирования драйверов

---

## ⚠️ Важные замечания

- Драйвер тестовый и предназначен только для образовательных целей
- Для запуска требуется **тестовый режим Windows** (TESTSIGNING ON)
- Тестирование производилось в изолированной виртуальной машине

---

* — Дальневосточный Федеральный Университет FEFU - , 2026*
