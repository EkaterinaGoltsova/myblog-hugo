+++
categories = [
	"Web-servers"
]
date = "2017-06-03T22:21:25+07:00"
title = "10 отличий Apache от Nginx"

+++

![Images](https://ekaterinagoltsova.github.io/img/nginx/nginx-vs-apache.png)

Доброго времени суток, backend/frontend/full-stack/devops/qa/.. да какая разница, добро пожаловать, мой друг!
	
Свой цикл статей хотелось бы начать с до неприличия банальной фразы - «Все познается в сравнении». Ведь невозможно говорить о том, что какой-либо инструмент лучший, не опробовав другой. Попробуем сравнить два самых популярных в мире веб-сервера - Apache, который обслуживает около 60 млн сайтов, и Nginx — около 40 млн (больше интересной статистики [тут](https://www.similartech.com/compare/apache-vs-nginx)). Возможно, после прочтения данной статьи вы сможете определиться, что лучше подходит для вашего dev-окружения. Итак, поехали! ;)

Начнем с архитектурных и функциональных отличий.
***
#### 1. Метод обработки соединений с клиентами

Издавна, **Apache** на каждый запрос от клиента создает отдельный процесс (или поток, зависит от выбранного mpm модуля). Выглядит это следующим образом - клиент отправляет запрос, веб-сервер создает отдельный процесс на этот запрос, отвечает клиенту и блокирует процесс до тех пор, пока клиент не закроет соединение. Это легко и просто в реализации, дебаге и мониторниге, но … Как вы могли бы догадаться, если у вас highload проект, то.. дела плохи. Процесс в любой ОС требует памяти и ресурсов, а когда процессов становиться неприлично много, обработка соединений неприлично  замедляется, память кончается, CPU растет. Для мелких проектов такая реализация архитектуры обработки соединений не добавит головной боли, но для высоконагруженных проектов придется ставить очень мощное железо или искать альтернативные варианты. 

**Nginx** состоит из master-процесса и нескольких дочерних процессов. Мастер процесс обычно один — он создает дочерние процессы (воркеры, загрузчик кеша и кеш менеджер), считывает конфигурацию и открывает порты. Воркеров обычно несколько, разработчики nginx советуют количество воркеров определять равным числу ядер машины. Эти дочерние процессы буду обслуживать все соединения с клиентами в неблокирующей манере. В nginx используется бесконечный цикл, который бежит по всем соединениями и отвечает на запросы клиентов. Когда соединение закрывается, оно удаляется из event loop. Это решение идеально подходит для проектов, которые обслуживающих 10к+ соединений одновременно. При этом, загрузка CPU и использование памяти обычно равномерны, без видимых пиков.
*** 
#### 2. Отдаваемый контент

**Apache** — может генерировать как статический контент, так и динамический. С этим никаких проблем нет. Прекрасно подойдет тем, кто не хочет заморачиваться с проксированием и настройкой дополнительного инструмента для генерации динамики, ведь Apache — это  готовое работающее решение. 

**Nginx** — отдает только статику и из коробки генерировать динамический контент не умеет. Если вы используете nginx и хотите генерировать динамический контент на своем сайте, то вам придется проксировать запросы тому, кто это делать умеет (apache, php-fpm и др.). Поэтому, разработчикам придется настраивать дополнительную свзяку, которая усложняет архитектуру, например nginx+apache (кстати в этой связке, Apache назвают бекенд сервером, а Nginx — фронтендом), nginx + phpfpm, nginx + python и др. 
***
#### 3. Конфигурирование
 
**Apache** полюбился разработчикам и сисадминам не в последнюю очередь из-за возможности конфигурировать обработку соединений на уровне директорий.  Делается это с помощью скрытого файла .htaccess, позволяющего настраивать права доступа, авторизацию, аутентификацию, политику кеширования и др правила.  Это довольно-таки удобное решение для пользователей, потому что позволяет менять конфигурацию на лету, без перезагрузки сервера и без наличия доступа к основному конфигу сервера. Но также имеется  маленький минус — Apache каждый раз при обработке соединений ищет файл .htacces и считывает с него информацию, что естественно замедляет выдачу ответа клиенту (кстати, поддержку настройки конфигурирования на уровне директорий можно и отключить).

**Nginx** не поддерживает конфигурирование на уровне каталогов. Существует один конфигурационный файл на весь проект, который обрабатывает master. Если вы хотите обновить конфигурацию, то необходимо отправить сигнал `SIGHUP` мастеру, который в свою очередь перезагружает конфигурацию и плавно завершает работу воркеров.
***
#### 4. Работа с модулями

**Apache** за долгое время существования обзавелся около 60 официальными модулями, и еще большим числом неофициальных. Модули динамически подключаются, не требуют сборки и перезагрузки веб-сервера. 

**Nginx** имеет около 130 официальных [модулей](https://www.nginx.com/resources/wiki/modules/). В отличии от Apache, модули Nginx не могут быть динамически загружены на лету и требуют сборки. Это гораздно сложнее, но считается безопаснее. 
***
#### 5. Интерпритация запросов

**Apache** имеет возможность интерпретировать запрос как физический ресурс в файловой системе или как URI, который требует дополнительной обработки.

**Nginx** создан, чтобы работать и в качестве веб-сервера, и в качестве прокси-сервера. По этой причине он работает в первую очередь с URI, транслируя их при необходимости в запросы к файловой системе. 
***
#### 6. Работа со скриптовыми языками

В **Apache** есть один модуль `mod_php` и все хосты вынуждены работать с одной и той же версией php и одним конфигурационным файлом. 

В случае с **nginx**, каждый виртуалхост будет выполняться в отдельном процессе и, соотстветственно, может использовать разные версии php (python/ruby/perl и др.).  Каждый процесс может иметь свою собственную независимую конфигурацию. 

Вообще, в высоконагруженных проектах удобнее держать раздельно nginx и php. По отдельности их проще мониторить, ловить баги или узкие места. «Все-в-одном» Apache+mod_php в этом плане менее удобен.


От основных архитектурных и функциональных отличий переходим к показательным отличиям, отличиям инфрастктуры и всего того, что также является не менее важным при выборе веб-сервера.
***
#### 7.  Скорость работы 

Скорость работы веб-сервера обычно измеряют для 2-х случаев отдачи контента: для статики и динамики. 
На основе тестов производительности, **Nginx** примерно в 2.5 раза быстрее отдает статику, чем **Apache**. Это довольно-таки большое превосходство. Если вам необходимо обслуживать большое количество статического контента, Nginx — лучший выбор. 
Во време тестирования отдачи динамического контента, Apache и Nginx показывают примерно одинаковые результаты. С точки зрения памяти, оба сервера используют один и тот же объем ресурсов. 
(Подробнее о тестах скорости отдачи контента можно почитать [здесь](http://www.speedemy.com/apache-vs-nginx-2015/))
***
#### 8. Поддержка ОС

**Apache** прекрасно работает на Unix-подобных операционных системах, также разработчики этого веб-сервера полностью поддерживают линейку Microsoft Windows, включая последние версии этой ОС. 

**Nginx** также поддерживает работу на множестве Unix-подобных ОС и имеет некоторую поддержку Windows, которая не является полной. Но разве кто-то в наше время размещает веб-сервер на Windows? 
***
#### 9. Сообщество и поддержка

**Apache** на рынке с 1995 года, что очень немалый срок, обеспечивший инструменту огромное сообщество и поддержку с его стороны. Практически на все вопросы на Stack Overflow уже есть исчерпывающие ответы. Коммерческой поддержки нет. 

**Nginx** веб-сервер более молодой, на рынке он с 2004 года, что также не помешало большому сообществу сформироваться и поддерживать друг друга. Nginx, в отличии от Apache, имеет коммерческую версию Nginx Plus, которая дополнена инструментами балансировки нагрузки, мониторинга, потоковой передачи медиа и др. 
***
#### 10. Документация и обучение

И у **Apache** и у **Nginx** присутствует доступная официальная документация. 

Nginx предлагает платное обучение, включающее в себя онлайн курсы, практические занятия и экзамен. По окончании курса все участники получают сертификаты. Например, сдать экзамен по основам nginx и получить официальный сертификат обойдется в 49$ (подробнее [здесь](https://university.nginx.com/)). 
***
От себя хотелось бы отметить, что оба решения очень стабильны, безопасны и поддерживаемы. Выбирайте то, что подходит именно вам.  Пробуйте, эксперементируйте, ошибайтесь и снова пробуйте. Всем добра и будьте здоровы!

При написании статьи были использованы некоторые материалы ресурса [http://www.hostingadvice.com/how-to/nginx-vs-apache](http://www.hostingadvice.com/how-to/nginx-vs-apache)/	
