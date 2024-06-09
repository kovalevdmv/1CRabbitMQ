# 1CRabbitMQ
Внешняя компонента RabbitMQ для 1C

Нативная внешняя компонента написанная на Rust/C++ для удобного взаимодействия с брокером сообщений RabbitMQ без реализации внешних прокси сервисов.

Работает на Windows и Linux.

Реализованы следующие функции: подключение к серверу, создание канала, создание очереди, привязка очереди к точке обмена и указание ключа маршрутизации, публикация сообщения (с получением подтверждения от сервера, что повышает надежность обмена), создание получателя сообщений, получение сообщений с возможность подтвердить полученное сообщение (что снижает риск потери сообщений).

Основная логика написана на Rust, что снижает вероятность утечки памяти, по сравнению с разработкой на С++, но при этом дает быстродействие сопоставимое с приложениями на С++.

Версии платформы: В linux ограничений нет, в windows требуется версия от 8.3.21 и выше

Компонента поставляется в двух формах, в платной и бесплатной. У бесплатной версии есть одно ограничение, на размер сообщения - 10000 символов. Компонента поставляется в виде конфигурации с примерами и обработкой КлиентRMQ, которая является оберткой над компонентой, для более удобной работы с ее методами из кода. Для использования необходимо скопировать обработку КлиентRMQ в конфигурацию, вся работа реализуется через эту обработку.

# Текущая версия 0.9.5
Скачать можно [по ссылке](https://1ctools.website.yandexcloud.net/RMQClientFree.7z)

# Примеры кода
## Отправка
```
//Создать экзезмпляр обработки-клиента RMQ
КлиентRMQ = Обработки.КлиентRMQ.Создать();
// Выполнить подключение к серверу
Подключение = КлиентRMQ.ПодключитьсяКСерверу("amqp://guest:guest@127.0.0.1:5672/%2f");
// Создать канал в рамках подключения. 
//В канале указать что нам нужно получать подтверждение от сервера при публикации сообщений
Канал = КлиентRMQ.СоздатьКанал(Подключение, Истина);
// Опубликовать сообщение с указанием ключа маршрутизации и точки маршрута
Ответ = КлиентRMQ.ОпубликоватьСообщение(Канал, "text message", "test", КлиентRMQ.Exchange_direct());
// !!! ОБЯЗАТЕЛЬНО ОТКЛЮЧИТЬСЯ ОТ СЕРВЕРА. ИНАЧЕ ЛИШНИЙ РАСХОД ПАМЯТИ.
КлиентRMQ.ОтключитьсяОтСервера(Подключение);

```
## Получение
```
// Создать экзезмпляр обработки-клиента RMQ
КлиентRMQ = Обработки.КлиентRMQ.Создать();
// Выполнить подключение к серверу
Подключение = КлиентRMQ.ПодключитьсяКСерверу("amqp://guest:guest@127.0.0.1:5672/%2f");
// Создать канал в рамках подключения.
Канал = КлиентRMQ.СоздатьКанал(Подключение);
// Создать получателя, через которого будут получаться сообщения из очереди
Получатель = КлиентRMQ.СоздатьПолучателя(Канал, "test");
// СледующееСообщение() ожидает сообщение в очереди. 
//Эта функция блокирует основной поток, поэтому этот код надо запускать в фоне.
Пока КлиентRMQ.СледующееСообщение(Получатель) Цикл // только ожидает сообщение
	// получить данные сообщения
	ДанныеСообщения = КлиентRMQ.ДанныеСообщения(); // получает данные сообщения
	Сообщить(ДанныеСообщения.Данные);
	// подтвердить получение сообщения. После чего оно удалится из очереди.
	Ответ = КлиентRMQ.ПодтвердитьСообщение(Канал, ДанныеСообщения.Тег);
	Прервать;
КонецЦикла;
// !!! ОБЯЗАТЕЛЬНО ОТКЛЮЧИТЬСЯ ОТ СЕРВЕРА. ИНАЧЕ ЛИШНИЙ РАСХОД ПАМЯТИ.
КлиентRMQ.ОтключитьсяОтСервера(Подключение);
```
