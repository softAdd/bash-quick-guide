# Основы bash-скриптов

## Что это такое?

Bash-скрипты - это сценарии командной строки, написанные для оболочки bash.<br>
Сценарии командной строки позволяют исполнять те же команды, которые можно ввести<br>
в интерактивном режиме.<br>
Интерактивный режим - оболочка читает команды, вводимые пользователем. Пользователь имеет возможность<br>
взаимодействия с оболочкой.

## Создание скриптов

Для того, чтобы создать bash-скрипт, необходимо создать файл с расширением *.sh<br>
Первой строкой необходимо указать, что этот файл является исполняемым командной оболочкой bash:<br>
`#!/bin/bash`

## Исполнение скриптов

Существует 2 основных способа вызвать bash-скрипт:<br>
1) `bash *.sh`<br>
2) `./*sh`

Для того, чтобы воспользоваться вторым способом, необходимо сначала выдать файлу правильные права:<br>
`sudo chmod +x *.sh`

## Работа с переменными внутри скриптов

Создание простой переменной со значением (обязательно без пробелов):<br>
`myVariable="test"`

Вернуть в переменную значение команды bash:<br>
```bash
myOs=`uname -a`
```
или<br>
```bash
myOs=$(uname -a)
```

Использование переменной в тексте:<br>
`echo "$myOs"`

Использование переменной с командами:
`echo $myOs`

Пример простой работы с переменными:<br>
```bash
num1=50
num2=45
sum=$((num1+num2))
echo "$num1 + $num2 = $sum"
```

## Параметры скриптов

Параметры скриптов - значения, переданные в файл при его вызове, например:<br>
`echo text` - вызов команды echo с параметром text.

Внутри bash-скриптов можно обращаться к специальным переменным:<br>
`$0` - хранит в себе название файла.<br>
`$` + любая цифра - переданный файлу параметр.

Пример использования:<br>
`bash *.sh hello`

*.sh:<br>
`echo $1`

Результат:<br>
`hello`

## Условные операторы

### if-then

```bash
user=someUsername
if grep $user /etc/passwd
then
echo "The user $user Exists"
fi
```

### if-then-else

```bash
user=someUsername
if grep $user /etc/passwd
then
echo "The user $user Exists"
else
echo "The user $user doesn’t exist"
fi
```

### elif

```bash
user=someUsername
if grep $user /etc/passwd
then
echo "The user $user Exists"
elif ls /home
then
echo "The user doesn’t exist but anyway there is a directory under /home"
fi
```

## Перенос строки в bash

С помозью слеша можно переносить продолжение команды на новую строку:
```bash
yaourt -S \
package1 \
packege2 \
package3
```

## Циклы в bash-скриптах

### for

Самый простой вариант цикла:<br>
```bash
for var in "the first" second "the third"
do
echo "This is: $var"
done
```

Цикл из результата работы команды:<br>
```bash
file="myfile"
for var in $(cat $file)
do
echo " $var"
done
```

Цикл из результата работы команды с разделителем полей:<br>
```bash
file="/etc/passwd"
IFS=$'\n'
for var in $(cat $file)
do
echo " $var"
done
```

IFS (Internal Field Separator) - специальная переменная окружения, которая позволяет указывать разделители полей.<br>
По умолчанию bash считает разделителями следующие символы: пробел, знак табуляции, знак перевода строки.

### for в стиле C

```bash
for (( i=1; i <= 10; i++ ))
do
echo "number is $i"
done
```

### while

```bash
var1=5
while [ $var1 -gt 0 ]
do
echo $var1
var1=$[ $var1 - 1 ]
done
```

### Обработка вывода в цикле

```bash
for (( a = 1; a < 10; a++ ))
do
echo "Number is $a"
done > myfile.txt
echo "finished."
```

С помощью символа ">" можно куда-нибудь перенаправить вывод, например в файл.<br>
В данном примере оболочка создаст файл myFile.txt и перенаправит в него вывод.

## Запуск bash-скриптов вместе с системой

Раньше было принято размещать все скрипты, которые запускаются по умолчанию в файле /etc/rc.local.<br>
Этот файл все еще существует, но это пережиток системы инициализации SysVinit и теперь он сохраняется только для совместимости.<br>
Скрипты же нужно загружать только с помощью Systemd.

Для этого достаточно создать простой юнит-файл и добавить его в автозагрузку, как любой другой сервис.<br>
Сначала создадим этот файл:<br>
`sudo vi /lib/systemd/system/runscript.service`<br>
Добавим содержимое:<br>
[Unit]<br>
Description=My Script Service<br>
After=multi-user.target

[Service]<br>
Type=idle<br>
ExecStart=/usr/bin/local/script.sh

[Install]<br>
WantedBy=multi-user.target

В секции Unit мы даем краткое описание нашему файлу и говорим с помощью опции After,<br>
что нужно запускать этот скрипт в многопользовательском режиме (multi-user).

Секция Service самая важная, здесь мы указываем тип сервиса - idle, это значит, что нужно просто запустить и забыть,<br>
вести наблюдение нет необходимости, а затем в параметре ExecStart указываем полный путь к нашему скрипту.

Осталось выставить правильные права:<br>
`sudo chmod 644 /lib/systemd/system/runscript.service`

Затем обновить конфигурацию и добавить в автозагрузку Linux новый скрипт:

`sudo systemctl daemon-reload`<br>
`sudo systemctl enable myscript.service`

После следующей перезагрузки этот скрипт будет запущен автоматически. Обратите внимание, что для каждого скрипта,<br>
который вы собираетесь запускать должны быть правильно выставлены права, а именно нужно установить флаг выполнения. Для этого используйте команду chmod:

`sudo chmod u+x /usr/local/bin/script`<br>
В параметрах мы передаем утилите адрес файла скрипта. Исполняемость - это обязательный параметр для всех способов.