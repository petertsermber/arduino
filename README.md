Здравствуйте.
Начал осваивать МЖД. Купил ардуино, датчики 18B20. Загрузил скетч (http://smartliving.ru/Main/ArduinoSchema).
Для освоения скетч хороший. Но меня не устраивало, что при подключении нового датчика, постоянно требовалось вносить изменения в код и перезаливать его в плату Arduino (перед этим отключив все шилды), что, согласитесь, крайне не удобно.

Решил написать универсальный скетч (который бы и команды на реле посылал, и всевозможные датчики бы контролировал, а самое главное - чтоб умел самостоятельно найти, определить и отсылать информацию на сервер MJD).

Представляю Вашему вниманию Universal Arduino sketch for MajorDoMo: https://github.com/petertsermber/arduino/blob/master/Universal%20Arduino%20sketch%20for%20MajorDoMo.ino

Пока работает только с датчиками 18B20 (но я в ближайшем будущем добавлю поддержку распространенных датчиков: DHT (влажности и температуры), BPM085 (давления и температуры, PIR (датчик движения), а так же модуля управления реле на 4 канала).

Ардуино ищет подключенные с 1-wire шине датчики, определяет их адреса, заносит в массив.
Далее по циклу, с каждого датчика получаем температуру, округляем ее до десятых, сравниваем с прошлой отправленной на сервер температурой. Если дельта больше 0,3 градуса, то отправляем данные на сервер МЖД. Идентификатором на сервере служит адрес датчика в HEX формате. Затем цикл повторяется.
Для "горячего" подключения датчиков требуется всего лишь "железно" подцепить датчик к шине, а в МажорДоМо добавить новый объект с именем типа 28156B15060000A7 (Адрес нового датчика можно узнать в стандартной программе типа Hyper Terminal), в котором существует свойство temp. Ну и должен присутствовать стандартный метод tempSensors-> tempChanged (который присутствует в стандартном дистрибутиве MJD). ВСЕ! Никаких перепрограммирований, перепрошиваний и прочего!

P.S. Я не программист, си++ вообще впервые увидел три недели назад, если что в коде не так, прошу сильно не ругать, а здесь написать.

Спасибо.
