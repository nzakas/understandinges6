<div dir="rtl">

# הקדמה

תכונות השפה הקרויה   JavaScript  מוגדרת בתקן הקרוי   ECMA-262.  
שפת התכנות המוגדרת בתקן זה נקראת  ECMAScript. <BR/>
מה שמוכר לך כ  JavaScript  בדפדפנים וב  Node.js הוא למעשה הרחבות  של   ECMAScript. 
דפדפנים וסביבית Node.js מוסיפים פונקציונאליות  בעזרת אוביקטים נוספים ומטודות , אבל עדיין  ה core של השפה נשאר  <BR/>
ECMAScript. 
ההתפתחות הפרואקטיבית של סטנדרט  ECMA-262  חיונית להצלחתה של ה    JavaScript  כמכלול. <BR/>
ספר זה סוקר השינויים שנעשו בשפה  בעדכון החשוב המרכזי  : ECMAScript 6. 


## הדרך ל-  ECMAScript 6

ב 2007  JavaScript  היתה בצומת דרכים.   
הפופולאריות של  Ajax גברה  בעידן של אפליקציות ווב דינאמיות,   
בעוד   JavaScript לא השתנה מאז הגרסה השלישית   של התקן  ECMA-262  שפורסמה ב  1999.
TC-39, הועדה האחראית לקדם את התפתחות      ECMAScript  הכינה דראפט  גדול ל      ECMAScript 4  שהיה מאוד מאסיבי בכיסוי נושאים , 
ושהציג שינויים קטנים וגדולים בשפה. 
היכולות העדכניות כללו  סינטאקס חדש,   modules, classes,  classes inheritance. 
הסקופ הרחב של    ECMAScript 4  גרמה לשינוי דרמטי בועדה   TC-39 ,  כאשר מספר חברי ועדה שברו  שגרסה זו ניסתה להשיג יותר מדי.  
קבוצה של מנהיגים  מ    Yahoo, Google, and Microsoft  יצרו  דראפט אלטרנטיבי  עבור    ECMAScript  שבתחילה כונה   ECMAScript 3.1 שכלל  מעט שינויי סינטאקס , והתרכז בפרופרטי אטריביוטס  , תמיכה ב    native JSON  והוספת מטודות לאוביקטים קיימים .
למרות שהיה ניסיון מסויים לאמץ  את   ECMAScript 3.1    ו   ECMAScript 4  זה כשל בתומכי שתי המחנות  והיה קושי רב ל לגשר בין  שני הפרספקטיבות של שני המחנות לגבי הדרך שבה השפה צריכה להתפתח. 
בשנת 2008   Brendan Eich  היוצר של JavaScript  הכריז  שהועדה   TC-39   צריכה להתרכז ב   ECMAScript 3.1 . 
הוא טען שיש להמתין עם עיקר שינויי הסינטאקס   של  ECMAScript 4  לשלב מאוחר יותר,   ושכל חברי הועדה יעסקו  בהבאת   ECMAScript 3.1  ו ECMAScript 4  לתקן חדש   שכונה ECMAScript Harmony.  
לבסוף  ECMAScript 3.1  הוגדר כתקן גרסה 5   של    ECMA-262  וכונה ECMAScript 5. הועדה  לא פירסמה את ECMAScript 4 בכדי לא לגרום לבילבול . 
אז החלה הועדה לעסוק   ב   ECMAScript Harmony , ולבסוף פורסם כ   ECMAScript 6  כתקן עם רוח  "הרמונית".  
ECMAScript 6  הגיע למצב של     feature complete  במהלך שנת 2015  ופוסמה פורמאלית כ   "ECMAScript 2015" (אבל עדיין הכוונה ל   ECMAScript 6 השם המקובל בקרב תכנתים בעוךם).  
בתקן זה התכונות  כוללות  מנעד רחב של שינויים  החל מאוביקטים חדשים   , תבניות  ועד לשינויי סינטאקס . 
המלהיב ב  ECMAScript 6   הוא שהשינויים  בשפה מוכוונים היטב  לפתרון בעיות שמפתחים נתקלו בהם בפועל.

 
## על ספר זה 
הבנה טובה  של התכונות והיכולות של   ECMAScript 6   הוא מפתח לכל מפתחי ה  JavaScript המתקדמים לאימוץ התקנים החדשים. 
התכונות בשפה המוצגת ב    ECMAScript 6  מציגות הבסיס  עליו אפליקציות   JavaScript   יבנו בעתיד הנראה לעין. 
 למטרה זו נכתב ספר זה.  תקוותי שתקראו ספר זה למוד על התכונות והיכולות  של  ECMAScript 6  כך שתהיו מוכנים  להתחיל להשתמש בתכונות והיכולות החדשות בכל עת שתזקקו להם . 

 
### תאימות ל  Node.js ודפדפנים 
סביביות רבות של   JavaScript,  כדפדפנים שונים  ו  Node.js , עובדים על יישום בפועל  של   ECMAScript 6 . 
ספר זה אינו עוסק בבעיות יישום התקן בסביבות השונות  אלא מתרכז בהגדרות התקן  והתנהגות נכונה. 
לכן , יתכן שבסביבות מסויימות תתקלו בבעיות תאימות לתקן שלא נכללות בספר זה. 
 
### למי מיועד ספר זה 

ספר זה נועד להדריך אלו המכירים  JavaScript  ו ECMAScript 5.  בעוד שהבנת JavaScript לעומק אינה הכרחית לשימוש בספר זה ,  הוא יסיעע לכם להבין ההבדלים בין   ECMAScript 5  ו ECMAScript 6. 
במיוחד , ספר זה נועד  למעבר  מהיר לתכנות מתקדם  למפתחים בסביבת   Node.JS וקוד לדפדפנים המעונינים ללמוד על התפתחויות  עדכניות  בשפה וליישם אותם. 
ספר זה אינו למתחילים שמעולם לא תכנתו ב  JavaScript. הנכם אמורים להיות בעלי הבנה בסיסית טובה  בשפה בכדי להשתמש  בספר זה. 
 
 
###  תוכן ענינים 

כל אחד מ - 13 הפרקים בספר מכסה  אספקט אחר  ב   ECMAScript 6.  פרקים רבים מתחילים בדיון על  הבעיות ש  ECMAScript 6 נועדו לפתור ,   בכדי להציג יריעה רחבה לסיבות  ששינוים אלו הוגדרו.   
כל פרק מכיל מספר דוגמאות קוד לסייע לך ללמוד סינטאקס חדש וקונספטים חדשים. 


<B>פרק  1 : כיצד עובד Block Binding </B>  הפרק מדבר על    `let` and `const` , החלפת block-level gcur  var. 

<B> פרק 2 : Strings ו Regular Expressions </B> הפרק סוקר פונקציונאליות נוספת חדשה לעיבוד ומניפולאציות על מחרוזות string ו בחינה  וכן מציג הקונספט החדש  template strings. 

 
<B> פרק 3 : Functions in ECMAScript 6 </B>  הפרק עוסק בשינויים לגבי פונקציות.   זה כולל  תבנית פונקציות חץ,  ערך ברירת מחדל לפרמטרים של פונקציה  ונושאים נוספים. 
  
<B>   פרק 4 :  הרחבת פונקציונאליות של אוביקטים   </B>  מסביר את השינוי כיצד אוביקטים נוצרים, עוברים שינוי וכיצד משתמשים בהם.  הנושאים כוללים שינוי בליטרל של אוביקט , ומטודות הצגה. 

<B>  פרק 5 : פעולת הרס אוביקטים  עבור גישה פשוטה יותר לנתונים </B>  מציג הרס אוביקטים ומערכים , המאפשר לבמע דקומפוז  של אוביקטים ומערכים בצורת כתיבה קצרה יותר. 

<B>  פרק 6 :  Symbols ו - Symbols property  </B>  מציג את הקונספט של Symbols, דרך חדשה להגדיר תכונות אוביקט. Symbols הוא סוג חדש של  primitive type שניתן להשתמש בו  בצורה מוסתרת (אבל לא נסתרת לחלוטין )  לגבי תכונות   ומטודות  של  אוביקט . 

<B>  פרק 7 : Sets ו Maps   </B>  מפרט את הסוגים החדשים  של   `Set`, `WeakSet`, `Map`, and `WeakMap`.  סוגים חדשים אלו מרחיבים את השימושיות של מערך  , מבחינת סמנטיקה ת עיבוד ת ניהול זיכרון שהותאם במיוחד ל  JavaScript. 
 
<B>  פרק 8 : Iterators ו Generators  </B>  עוסק בתוספת של   Iterators ו Generators  לשפה. , תכונות אלו מאפשרות לך לעבוד עם אוספים של דאטה  בצורה יעילה ועוצמתית  שלא התאפשרה בגרסה הקודמת של  JavaScript. 

<B> פרק 9 : הצגת   Classes ב JavaScript  </B>  מציג  הקונספט הפורמאלי הראשון  של  Classes ב JavaScript. לעיתים נקודת בילבול בין מה שמקיים בשפות  תכנות אחרות , התוספת של סינטאקס  של  Class ב JavaScript מאפשר שימוש בשפה בצורה יותר  נוחה ובצורה  מקוצרת  למי שמעוניין. 
 
<B> פרק 10 : יכולות משופרות של Array  </B>  מפרט השינויים  למערכים  רגילים ודרכים מענינות חדשות בהם מערכים  ניתנים לשימוש ב JavaScript.  

<B>  פרק 11 : Promises ותכנות אסינכרוני </B>  מציג את Promises כחלק חדש בשפה.   Promises היו מאמץ שורשי  לשינוי שבסוף הצליחו להיות פופולארים בזכות תמיכה מאסיבית בספריות .    ECMAScript 6  מציגה Promises בצורה פורמאלית ומאפשרת זמינותם כברירת מחדל. 

<B>  פרק 12 :  Proxies וה  API  של ה Reflection </B>  מציג הצורה הפורמאלית של  reflection API בשפת JavaScript ואת ה proxy object החדש שמאפשר למפתח  לתפוס כל אירוע  של שינוי Object. Proxies מאפשרים למפתח שליטה ללא עוררין על  ה Objects , וככאלה אפשרויות אינסופיות של תבניות שליטה אינטראקטיביות . 

<B>  פרק 13 : עטיפת קוד בעזרת Modules </B> מפרט את ה Module פורמט  ב JavaScrupt.  המוטיב הוא שמודולים אלו יכולים להחליף מספר רב של ad-hoc module definition format שהופיע ברבות השנים. 
 
<B> Appendix A:  שינויים קטנים ב  ECMAScript 6 </B>  סוקר שינויים ב  ECMAScript 6   שהם בשימוש פחות שוטף ולכן לא בוסו בפרקים אחרים. 

<B> Appendix B:  הבנת  ECMAScript 7  </B>  מתאר את שתי התוספות לתקן ב  ECMAScript 7 , שלא השפיעו על JavaScript כמו ECMAScript 6  




</div>



<div dir="rtl">

### סימונים מוסכמים שבשימוש בספר זה 

להלן הסימונים הטיפוגראפיים  והמוסכמות בשימוש בספר זה 

* *Italics* מסמנות הגדרה חדשה 
* `Constant width` מסמן קטע קוד או שם קובץ

בנוסף, קטעי דוגמת קוד ארוכים מוכלים במסגרות  בלוק של קוד כדוגמת : 

```js
function doSomething() {
    // empty
}
```

בתוך הקוד בלוק, סימני הערה  עם גרשיים בצד ימין מציגים את הפלט שיוצג בדפדפן  בקונסול או ב Node.js בקונסול  לאחר הרצת הקוד, לדוגמא  לפקודה console.log : 

```js
console.log("Hi");      // "Hi"
```

במידה ושורת קוד תיצור חריגה גם  זה יצויים בצד ימין בקומנט כגון 

```js
doSomething();          // error!
```

### עזרה ותמיכה 

תוכלו להעלות נושאים , הצעות שיפור או  פול רקוואסט  לספר זה על ידי  גלישה לכתובות 

[https://github.com/nzakas/understandinges6](https://github.com/nzakas/understandinges6)


</div>



<div dir="rtl">

###  תודות לתורמים לספר 

</div>



Thanks to Jennifer Griffith-Delgado, Alison Law, and everyone at No Starch Press for their support and help with this book. Their understanding and patience as my productivity slowed to a crawl during my extended illness is something I will never forget.

I'm grateful for the watchful eye of Juriy Zaytsev as tech editor and to Dr. Axel Rauschmayer for his feedback and several conversations that helped to clarify some of the concepts discussed in this book.

Thanks to everyone who submitted fixes to the version of this book that is hosted on GitHub: ShMcK, Ronen Elster, Rick Waldron, blacktail, Paul Salaets, Lonniebiz, Igor Skuhar, jakub-g, David Chang, Kevin Sweeney, Kyle Simpson, Peter Bakondy, Philip Borisov, Shaun Hickson, Steven Foote, kavun, Dan Kielp, Darren Huskie, Jakub Narębski, Jamund Ferguson, Josh Lubaway, Marián Rusnák, Nikolas Poniros, Robin Pokorný, Roman Lo, Yang Su, alexyans, robertd, 404, Aaron Dandy, AbdulFattah Popoola, Adam Richeimer, Ahmad Ali, Aleksandar Djindjic, Arjunkumar, Ben Regenspan, Carlo Costantini, Dmitri Suvorov, Kyle Pollock, Mallory, Erik Sundahl, Ethan Brown, Eugene Zubarev, Francesco Pongiluppi, Jake Champion, Jeremy Caney, Joe Eames, Juriy Zaytsev, Kale Worsley, Kevin Lozandier, Lewis Ellis, Mohsen Azimi, Navaneeth Kesavan, Nick Bottomley, Niels Dequeker, Pahlevi Fikri Auliya, Prayag Verma, Raj Anand, Ross Gerbasi, Roy Ling, Sarbbottam Bandyopadhyay, and Shidhin.

Also, thanks to everyone who supported this book on Patreon: Casey Visco.




