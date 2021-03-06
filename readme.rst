Записки говнокодера
===================


Общие вопросы по бэкендам
-------------------------


Тесты
-----

* Всегда пишите юнит тесты. Не важно ТDD или какое другое сокращание. Пишите, блядь, тесты. Даже если вы пишете на C++, тоже пишите тесты. Это нужно вам, чтобы быть уверенными, что вы не просто так получаете зарплату.

* Следите за coverage ваших тестов. Очень часто кажется что вы протестировали весь ваш код, но на практике тест тестирует не то что вам надо. Если ваш coverage меньше 80%, срочно бросайте всё и начинайте писать тесты. Если 90%, то вы молодец, но можно лучше.
Если у вас coverage > 95%, то либо одна из черепашек пиздит (с), либо у вас проект уровня hello world.

* Не забывайте писать тесты на bad flow. То есть пишите проверки на неправильные значения и различные ошибки. Об ошибках в good flow (когда данные валидны и всё идет хорошо) вы узнаете сразу, когда QA пойдет тыкать в вашу какашку. А вот проблемы в bad flow часто доезжают до продакшена и юзер получает 500ку вместо корректного сообщения что, что-то не так.

* Используйте pytest. Ничего лучшего его фикстур на данный момент не придумано.

* Если ваши юнит тесты идут больше 10 минут, это не тесты а издевательство над кодером. Всегда думайте как можно сделать тесты быстрее. 
  Медленные тесты никому не нужны. 
  
* Не используйте sleep() в тестах, это может быть очень медленно. Если нужно протестировать какой нибудь time effect, то лучше в функциях передавать текущее время как не обязательный параметр. Всякие хипстеры называют это Dependency Injection.

.. code-block:: python

      def foo(param1, param2, now=None):
          now = now or datetime.datetime()
          ...
          
          
      def test():
          now = datetime.datetime.now()
          foo(1, "b", now=now)
          # instead of waiting 1 second, just DI it
          foo(2, "c", now + datetime.timedelda(seconds=1))

* Теория про юниттесты, говорит что юниттесты надо делать по возможности изолированными от других тестов и внешней среды.
  Но если на половине тестов вам приходится заного восоздавать одну и туже среду (пусть и с мелкими доработками ньюансами), 
  то это bad practics, потому что ваши тесты будут делать много не нужной работы, а на нужную работу у вас не останется времени.
  Также в случае интеграционными тестами, у вас не будет шанса "обнулить базу". Учитесь писать тесты, которые бы учитывали, 
  что в базе уже есть мусор. Например, вместо проверки на абсолютные значения, проверяйте "дельты". То есть вместо

.. code-block:: python
   org = Organization.create()
   assert len(Organization.query.all()) == 1


напишите

.. code-block:: python
   total_orgs = len(Organization.query.all())
   org = Organization.create()
   assert len(Organization.query.all()) == total_orgs + 1
   

* Обязательно пишите клиент который может ходить в ваш же бэкенд. Пишите сами, не перекладывайте на QAщиком или juniorов. 
  Если придуманный вами интерфейс не удачный (говно), то уже на попытке написать клиента, вы это можете увидеть что ваш API говно и им нельзя пользваться. 

* Не забывайте писать интеграционные тесты, которые бы ходили в уже развернутый бэкенд / бэкенды, с использованием вашего же клиента. Обязательно пишите сами, не перекладывайте на других. Написав, хотя бы good flow cases, вы можете отдать свой клиент/API для тестов тест автоматчику, чтобы надругался над вами API и нашел все проблемы в коде. Ни один программист над своим кодом не будет так издеваться, как QA  automation с 10 летним стажем. 


Python
------

* Используйте ContextManager. Если у вас возникло желание что то сделать в `__del__`, то убейте себя "апстену", и подумайте что ContextManager будет более удачным решением. Жаль, мне это не вдолбили 10 лет назад. 


Протоколы
---------

- Если вы решили сделать свой RPC протокол, то есть аля команда и её аргументы, и этот протокол построен на 
  базе словарей/json/msgpack и т.д., 
  то передавайте параметры в отдельном словаре, то есть

.. code-block:: json

      {"command": "hello", "params": {"who": "kitty"}}

а не 

.. code-block:: json

      {"command": "hello", "who": "kitty"}

Кажется, что во втором случае экономится один словарик и он должен быть эффективным, но как правило у вас возникнут ситуации 
когда вы захотите выделить только параметры в виде словаря, например для логирования или для их валидации.

База данных
-----------

- primary_key в табличках надо называть как <table_name>_id, а не просто id. Например organization_id, user_id, shit_it. 
  Кажется что это лишняя трата буковок и усилий программиста по их копипасте, но как только у вас пойдут сложные 
  запросы с join'ами то это, возможно, поможет избежать сравнения разных айдишников, аля organization_id == shit_id. 
  Также имена foreign key полей будут такими же.

То есть вот так "есть хорошо"

.. code-block:: sql

   SELECT ***
   FROM
   WHERE organization.organization_id == user.organization_id

а вот так может получиться путаница в итоге, особенно когда условий будет много

.. code-block:: sql

   WHERE organization.id == user.organization_id


REST
----


Конфиги
-------
