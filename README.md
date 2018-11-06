
<meta charset="utf-8">


# Как ворваться в Data Science и получить видеокарту от Гугла

Вы начинающий Data Scientist, посмотревший лекции и разобравшийся с теорией, а теперь жаждущий попрактиковаться, но мощного железа под рукой нет? Что ж, с помощью инструментов компании Google вы можете начать учить ваши первые нейроные сети уже сейчас!



В этом туториале я расскажу и покажу, как настроить доступ к сервису Google Cloud Platform и получить мощную видеокарту на целых 10 Терафлопс!



## Шаг 1 - Найдите

Найдите сайт google cloud platform ![](tutorial/photo1.png)


Он выглядит как-то так. ![](tutorial/photo2.png)






## Шаг 2 - Зарегистрируйтесь

Зарегистрируйтесь с помощью акаунта Google.

![](tutorial/photo3.png)

Google предложит вам бесплатные 300$. 

![](tutorial/photo4.png)

Вот это же предложение подробнее. 

![](tutorial/photo5.png)

Вас попросят ввести данные вашей карточки. 

![](tutorial/photo6.png)

Если предложение от Гугла не вылезло, то заходим в раздел «Оплата», где сами создаём платёжный аккаунт. Да, тут точно также дадут 300$ на первое время.

![](tutorial/photo7.png)






## Шаг 3 - Создайте свое первый инстанс

Заходим в раздел «Compute Engine» -> Экземпляры ВМ 

![](tutorial/photo8.png)

Создаём проект

После чего заходим в раздел «Квоты», для того, чтобы запросить увеличение квоты по видеокартам. По умолчанию эта квота равна 0. 

![](tutorial/photo9.png)

Ищем квоту по видеокартам NVIDIA. Выбираем себе карточку. Для большинства задач подойдет Tesla K80, её дополнительный плюс в том, что она достаточно дешёвая. Но я брал себе Tesla P100, Just for fun.


Tesla P100 как раз таки имеет 10 Терафлопс, у K80 их около 2~3.

![](tutorial/photo10.png)

Далее внимательно выбираем себе регион, советую брать Европу или Азию, в них меньше пинг, в целом, помимо пинга отличий особо нет.


ВАЖНО! Перед тем, как запрашивать карточку в том или ином регионе надо проверить, доступна ли она вообще для данного региона.

Сделать это можно [тут](https://cloud.google.com/compute/docs/gpus/)

Доступные регионы случае с NVIDIA Tesla P100 

![](tutorial/photo11.png)

Выбираем квоту, нажимаем «Изменить квоты» в верхней части страницы. 

![](tutorial/photo12.png)

Формируем запрос и отправляем его, ответ придет в районе несколько часов. 

![](tutorial/photo13.png)

Выбираем «Создать экземпляр» 

![](tutorial/photo14.png)

Выбираем регион, где вам выделили видеокарту 

![](tutorial/photo15.png)

Жмём «Настроить» 

![](tutorial/photo16.png)

Мой дефолтный setup выглядит как-то так. 4 процессора и 16 Гб оперативки - лучше брать столько или слегка больше, эти ресурсы стоят не так дорого ( на порядок дешевле видеокарты), однако при их нехватке могут возникнуть серьезные подтормаживания. К слову, при тренировке resnet36 все 4 процессора работали на 70-80% и было занято 5~6 Гб оперативной памяти. 

![](tutorial/photo17.png)

Далее выбираем загрузочный диск. Советую брать Ubuntu LTS16, так как на неё более менее спокойно встает Cuda, с 18й же версией у меня возникали проблемы. По поводу размера не советую сразу брать много, так как за это постоянно списывают деньги, даже когда вы не используете ваш инстанс. 25 Гб для начала более чем достаточно 

![](tutorial/photo18.png)

Обязательно разрешаем HTTP и HTTPS траффик, чтобы была возможность подключения по ssh. Создаем экземпляр.

Вуаля, ваш инстанс готов, осталось подключиться к нему и настроить необходимые компоненты








## Шаг 4 - Подключение

Одним из вариантов подключения является протокол ssh, сейчас я объясню, как настроить ваш инстанс для работы по ssh.

Для начала генерируем пару ключей на вашем рабочем компьютере с помощью следующей комманды
``` 
ssh-keygen
``` 

Можно задать вашему ключу любое имя и хранить его в любом месте. Пункт «Passphrase» нужен для дополнительной защиты ключа, его можно пропустить. 

![](tutorial/photo19.png)

После генерации пары ключей нам необходимо закинуть публичный ключ на нашу виртуальную машину. А заходить на неё мы будем с помощью нашего приватного ключа.

Зайдем в папку, где мы сохранили ключи. 

![](tutorial/photo20.png)

«.pub» - публичный ключ, открываем его в любом текстовом редакторе и копируем содержимое.

![](tutorial/photo21.png)

Это будет выглядеть как-то так. 

![](tutorial/photo22.png)

После чего, нажимая на имя нашего инстанса, мы открываем настройку его параметров. Пролистывая вниз находим пункт «SSH-ключи». Вставляем публичный ключ сюда. При желании, изменив последнюю строчку публичного ключа до @ можно поменять имя пользователя. ![](tutorial/photo23.png)

Сохраняем изменения и запускаем виртуальную машину

У запущенной машины появится внешний ip адрес, запомним его. 

![](tutorial/photo24.png)

Далее, пишем в терминале следующую команду:
``` 
ssh -i path_to_your_private_key username@ip_addres
```

В моем случае это выгляди так:
``` 
ssh -i ~/.ssh/cuctom_key_name oak@35.204.41.62
```


При подключении могут возникнуть некоторые ошибки или непонятные проишествия, но все они легко гуглятся

Поздравляю вы подключились к вашему первому серверу!







## Шаг 5 - немного настройки

Если вы занимаетесь Data Science, то вам будет необходимо поставить Anaconda и Cuda

Anaconda установить достаточно просто, а как установить CUDA можно почитать [тут](https://developer.nvidia.com/cuda-90-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=debnetwork)

Если возникают проблемы с запуском jupyter notebook, то стоит установить его через pip и apt-get






## Шаг 6 - подключение к Jupyter Notebook через SSH и прокидывание портов

Для начала запустим Jupyter Notebook на удаленной машине. Как правило, он уже входит в пакет анаконды, но при необходимости его можно поставить через pip и apt-get.


Советую дополнительно поставить «Tmux» - эта утилита позволит вам запустить Jupyter notebook так, чтобы его работа не зависела от shh-сессии, что очень удобно


Пишем Tmux - у нас открывается дополнительный терминал, если его так можно назвать. Пишем следующий код:
``` 
jupyter notebook --no-browser --port=BBBB
``` 

«—no-browser» - так как мы запускаем на удаленной машине, без этого флага не запустится

«—port = BBBB» - указываем номер порта, желательно диапазон 8000 - 8100 (наверное, лучше уточнить), любой из диапазона на ваш вкус.


(если что, чтобы выйти из tmux нажмите CNTR + B и затем D)

Предварительно необходимо настроить пароль
``` 
jupyter notebook --generate-config
jupyter notebook password
``` 

Или можно скопировать токен, который сгенерирует jupyter notebook и затем вставить его в окошко пароля.

Затем нам необходимо прокинуть порт удаленной машины на нашу локальную машину.
``` 
ssh -i ~/.ssh/cuctom_key_name -N -f -L AAAA:localhost:BBBB username@server_ip
``` 

Как пример
``` 
ssh -i ~/.ssh/cuctom_key_name -N -f -L 8090:localhost:8080 fergus@server_ip
``` 

AAAA - порт на нашей локальной машине, желательно диапазон 8000 - 8100 (наверное, лучше уточнить), любой из диапазона на ваш вкус.

BBBB - порт, который мы указали при запуске jupyter notebook на удаленной машине

Затем заходим в браузер, где пишем localhost:AAAA

Вуаля, дерзайте!

(Перенос стиля, как один из use-case-ов применения нейронных сетей) 

![](tutorial/photo25.jpg)
