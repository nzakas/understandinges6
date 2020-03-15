<div dir="rtl">

# פרוקסי Proxies  ו  והשתקפות Reflection API

ECMAScript 5  ו ECMAScript 6 שניהם פותחו תוך חשיבה על הפשטה של פונקציות . לדוגמא, לפני ECMAScript 5 סביבת ג'אווה סקריפט הכילה מאפייני אובייקט בלתי ניתנים לגישה או לכתיבה לפני , ומפתחים לא יכלו להגדיר את האפשרויות של גישה וכתיבה למאפיינים שלהם. ECMAScript 5 כללה `Object.defineProperty()` מתודה  שמאפשרת למפתחים לעשות מה שהמנוע של ג'אווה סקריםט יכל לעשות כבר לפני.


ECMAScript 6 נתן למפתחים יותר גישה ליכולות של המנוע של ג'אווה סקריםט שלםפני כן היו רק כללות כבר באובייקטים. השפה חשפה  את הפעילות הפנימית של האובייקט דרך *proxies*, שהיא מעטפת שיכולה לגשת ולשנות פעולות ברמה הנמוכה של מנוע הג'אווה סקריפט. הפרק הזה יתחיל בתיאור הבעיה שפרוקסי בא לפתור ,ואז ידון איך ליצור פרוקסי ולהשתמש בו בצורה יעילה.

## בעית מערכים
אובייקט המערכים ב בג'אווה סקריםט מתנהג בדרכים שמפצחים לא יכלו לחקות על אובייקטים משלהם לפני ECMASCript 6. אורך מערך `length` המאפיין מושפע כרשר אתה עושה השמה של ערך למערך ספיציפי, ו , ואתה יכול לשענות את נתוני המערך על ידי שינוי הערך של `length` . לדוגמא:

</div>

```js
let colors = ["אדום", "ירוק", "כחול"];

console.log(colors.length);         // 3

colors[3] = "שחור";

console.log(colors.length);         // 4
console.log(colors[3]);             // "שחור"

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "ירוק"
```

<div dir="rtl">
מערך `colors` מתחיל עם שלושה אייטמים. השמת `"שחור"`  ל `colors[3]` אוטומטית מגדיל את המאפיין `length` ל `4`. מאפיין  השמה של  `length` ל `2` יסיר את שני האייטמים האחרונים במערך, וישאיר רק את השניים הראשונים. שןם דבר ב ECMAScript 5 מאפשר למפתחיל להשיג את היכולות הללו , אבל פרוקסי משנה את זה.

I> התנהגות לא סטנדרטית זו הסיבה שמערכים נחשבים לאובייקטים אקזוטיים ב ECMAScript 6.

## מה זה Proxies- פרוקסי ו Reflection - השתקפות?

אתה יכול ליצור פרוקסי שישתמש במיקום של אובייקט אחר (נקרא *target* *מטרה*) על ידי קריאה ל  `new Proxy()`. הפרוקסי *virtualizes* *ידמה* את המטרה כך שהפרוקסי והמטרה יופיעו באותו אובייקט והפונקצונאליות תהיה באמצעות הפרוקסי.

פרוקסי מאפשרים ליירט פעולות אובייקט ברמה נמוכה על המטרה שאחרת הם מנוהלות רק ע"י מנוע הג'אווה סקריפט. פעולות ברמה נמוכה אלו יורטו באמצעות סוג של *מלכודת* שהיא הפונקציה המגיבה לפעולה ספציפית.

ה reflection API, אובייקט מיוצג על ידי  `Reflect` ,  זה אסיפה של מתודות שמנגישות את ההתנהגויות הדיפולטיביות של אותה רמה נמוכה שפרוקסי יכול לשכתב אותם. יש מטודת `Reflect` עבור כל מלכודת פרוקסי. המתודות האלו עם אותו שם ומעבירות את אותם ארגומנטים כמו מלכודת הפרוקסי שלהם . טבלה 11-1 מסכמת את ההתנהגויות.

</div>

{title="Table 11-1: Proxy traps in JavaScript"}
| Proxy Trap               | Overrides the Behavior Of | Default Behavior |
|--------------------------|---------------------------|------------------|
|`get`                     | Reading a property value  | `Reflect.get()` |
|`set`                     | Writing to a property     | `Reflect.set()` |
|`has`                     | The `in` operator         | `Reflect.has()` |
|`deleteProperty`          | The `delete` operator     | `Reflect.deleteProperty()` |
|`getPrototypeOf`          | `Object.getPrototypeOf()` | `Reflect.getPrototypeOf()` |
|`setPrototypeOf`          | `Object.setPrototypeOf()` | `Reflect.setPrototypeOf()` |
|`isExtensible`            | `Object.isExtensible()`   | `Reflect.isExtensible()` |
|`preventExtensions`       | `Object.preventExtensions()` | `Reflect.preventExtensions()` |
|`getOwnPropertyDescriptor`| `Object.getOwnPropertyDescriptor()` | `Reflect.getOwnPropertyDescriptor()` |
|`defineProperty`          | `Object.defineProperty()` | `Reflect.defineProperty` |
|`ownKeys`                 | `Object.keys`, `Object.getOwnPropertyNames()`, `Object.getOwnPropertySymbols()` | `Reflect.ownKeys()` |
|`apply`                   | Calling a function | `Reflect.apply()` |
|`construct`               | Calling a function with `new` | `Reflect.construct()` |



<div dir="rtl">

כל מלכודת עוקפת התנהגות מובנית באובייקט של ג'אווה סקריפט, מאפשרת לך ליירט ולשנות את התתנהגות.אם אתה עדיין צריך את ההתנהגות הדיפולטיבית ,אתה יכול להשתמש במתודה התואמת ל  reflection API. היחסים בין פרוקסי ל reflection API נהיים ברורים שאתה מתחיל להשתמש בפרוקסי, ולכן עדיף לצלול פנימה ולהסתכל על כמה דוגמאות.

I>במפרט המקור של ECMAScript 6 הייתה מלכודת נוספת שנקראה `enumerate`  שנועדה לשנות את האופן שבו מתבצעת  הספירה`for-in` ו`Object.keys()` על האובייקט.אבל המלכודת של `enumerate` הוסרה ב ECMAScript 7 (שנקראית גם ECMAScript 2016) מכיוון שהתגלו קשיים במהלך היישום. מלכודת ה `enumerate` אינה קיימת עוד בשום סביבת JavaScript ולכן אינה מכוסה בפרק זה.

## לייצר Proxy פשוט

כאשר אתה משתמש ב `Proxy` constructor לייצר פרוקסי, אתה מעביר 2 ארגומנטים: את ה target ואת ה handler. ה *handler* הוא אובייקט אשר מגדיר מלכודת אחת או יותר . ההפרוקסי משתמש בהתנהגיות הדיפולטיביות חוץ מהמלכודותאשר הגדרת לפעולה מסויימת. בשביל לייצר פרוקסי העברה פשוט אפשר להשתמש ב handler בלי מלכודות:
</div>

```js
let target = {};

let proxy = new Proxy(target, {});

proxy.name = "proxy";
console.log(proxy.name);        // "proxy"
console.log(target.name);       // "proxy"

target.name = "target";
console.log(proxy.name);        // "target"
console.log(target.name);       // "target"
```

<div dir="rtl">

בדוגמא הזו, `proxy` מעביר את כל הפעולות ישירות ל   `target`. כאשר `"proxy"` עושה השמה למאפיין `proxy.name` , `name` נוצר על `target`. הפרוקסי עצמו לא מאחסן את המאפיין הזה ; הוא פשוט מעביר את המאפיין ל `target`. באופן דומה, הערכים של `proxy.name` ו `target.name` הם אותו הדבר בגלל שהם שניהם מיוחסים ל `target.name`. זה אומר שהגדרה חדשה ל `target.name` יגרום ח `proxy.name` להציג את אותם שינויים. כמובן, פרוקסי בלי מלכודות הוא לא מעניין, אז מה קורה כשאתה מגדיר מלכודת?

## אימות מאפיינים באמצעות מלכודת `set` 

נניח שברצונך ליצור אובייקט שערכי המאפיינים שלו חייבים להיות מספרים. המשמעות היא שיש לאמת כל מאפיין חדש שנוסף לאובייקט, ויש לזרוק שגיאה אם הערך אינו מספר. על מנת להשיג זאת, אתה יכול להגדיר מלכודת `set` אשר עוקף את התנהגות ברירת המחדל של הגדרת ערך. המלכודת `set` מקבלת 4 אגומנטים:

1. `trapTarget` - האובייקט שיקבל את המאפיינים (המטרה של הפרוקסי)
1. `key` - מפתח המאפיין (סטרינג או symbol) לכתוב אליו
1. `value` - הערך שנכתב למאפיין
1. `receiver` - האובייקט עליו התבצע הפעולה (בדר"כ הפרוקסי)

`Reflect.set()` הוא המתודה המקבילה ל `set` באובייקט ההשתקפות, וזו התנהגות ברירת המחדל עבור פעולה זו. המתודה `Reflect.set()`  מקבל את אותם ארבע ארגומנטים כמו  `set` שנמצא במלכודת הפרוקסי, מה שהופך את המתודה לקלה לשימוש בתוך המלכודת. המלכודת אמורה להחזיר  `true` אם המאפין נקבע ו `false` אם לא. (המתודה `Reflect.set()`  מחזירה את הערך הנכון על סמך האם הפעולה הצליחה.)

כדי לאמת את ערכי המאפיינים, ההית משתמש במלכודת `set` ובודק את `value` שמועבר בפנים. הנה דוגמא:
</div>

```js
let target = {
    name: "target"
};

let proxy = new Proxy(target, {
    set(trapTarget, key, value, receiver) {

        // התעלם ממאפיינים קיימים כדי לא להשפיע עליהם
        if (!trapTarget.hasOwnProperty(key)) {
            if (isNaN(value)) {
                throw new TypeError("Property must be a number.");
            }
        }

        // הוסף את המאפיין
        return Reflect.set(trapTarget, key, value, receiver);
    }
});

// הוספה של מאפיין חדש
proxy.count = 1;
console.log(proxy.count);       // 1
console.log(target.count);      // 1

// אתה יכול להקצות לשם מכיוון שהוא קיים כבר ביעד...
proxy.name = "proxy";
console.log(proxy.name);        // "proxy"
console.log(target.name);       // "proxy"

// יזרוק שגיאה
proxy.anotherName = "proxy";
```

<div dir="rtl">

קוד זה מגדיר מלכודת פרוקסי המאמתת את הערך של כל מאפיין חדש שנוסף ל `target`. כאשר `proxy.count = 1` מורץ, המלכודת `set` נקראית. הערך של `trapTarget` שווה ל `target`, `key` הוא `"count"`, `value` הוא `1`, ו `receiver` (שלא נעשה בו שימוש בדוגמא כאן) הוא `proxy`. לא קיים מאפיין כזה שנקרא `count` בתוך `target`, אז הפרוקסי מאמת את ה `value` ע"י העברה לפונקציה `isNaN()`. אם התוצאה היא `NaN`, אז המאפיין הוא לא מספר ושגיאה תיזרק. מאחר והקוד מקצה את `count` ל `1`, הפרוקסי קורא ל `Reflect.set()` עם אותם ארבע ארגומנטים שהועברו במלכודת בשביל להוסיף מאפיין חדש.

כאשר `proxy.name` הוא מוקצה לסנרינג, הפעולה מתבצעת בהצלחה. מאחר ול `target` קיים כבר המאפיין `name` , המאפיין הזה מושמט מבדיקת האימות שנקרית ע"י המתודה `trapTarget.hasOwnProperty()` . זה מבטיח שמאפיינים שלא נקבעו שהם לא מספרים עדיין נתמכים.

למרות זאת כאשר `proxy.anotherName` מוקצה לסטרינג,  שגיאה תיזרק. המאפיין  `anotherName` לא קיים על המטרה, לכן הערך שלו צריך לעבור אימות. בזמן האימות, שגיאה תיזרק כי `"proxy"` הוא לא ערך מספרי.

כאשר המלכודת הפרוקסי `set` מאפשרת לך ליירט כאשר נכתבים מאפיינים חדשים, המלכודת פרוקסי `get` מאפשרת לך ליירט כאשר מאפיינים נקראים.

## אימות צורת אובייקט באמצעות מלכודת `get`

אחד ההבטים המעניינים , ולעיתים מבלבלים, של  JavaScript כאשק קוראים למאפיינים שלא קיימים הוא לא זוקר שגיאה. במקום, הערך `undefined` הוא ממומש בשביל המאפיין, כמו בדוגמא:

</div>

```js
let target = {};

console.log(target.name);       // undefined
```

<div dir="rtl">

בשפות אחרות, נסיון לקרוא את `target.name` יזרוק שגיאה מאחר והמאפיין לא קיים.אבל ג'אווה סקריפט רק משתמש ב `undefined` בשביל הערך של המאפיין `target.name` . אם אי פעם עבדת על בסיס קוד גדול, בטח ראית כיצד התנהגות זו עלולה לגרום לבעיות משמעותיות, במיוחד כאשר יש שגיאת הקלדה בשם המאפיין. פרוקסי יכול לעזור לך להציל את עצמך מבעיה זו על ידי אימות צורת אובייקט.

ה *object shape* זה אוסף של מאפיינים ומתודות שנמצאים על האובייקט. מנוע הג'אווה סקריפט משתמש בצורת האובייקט לעשות אופטמיזציה לקוד ,לעיתים יוצר קלאס לייצג את האובייקט. אם אתה יכול להניח בבטחה שלאובייקט יהיו תמיד אותם מאפיינים והמתודות איתן הוא התחיל (התנהגות שאתה יכול להכריח ע"י המתודה `Object.preventExtensions()` , המתודה `Object.seal()` , או המתודה `Object.freeze()`), ואז זריקת שגיאה בניסיונות לגשת למאפיינים לא קיימות יכולה להועיל. פרוקסי מקל על אימות צורת האובייקט.

מאחר ואימות המאפיין קורה רק בזמן שאתה מנסה לקרוא את המאפיין, נצטרך להתשמש במלכודת `get` . מלכודת `get` נקרית גאשר מאפיין נקרא, אפילו אם המאפיין לא קיים על האובייקט, והוא מקבל 3 ארגומנטים:

1. `trapTarget` - האובייקט ממנו קוראים את המאפיין (פרוקסי target)
1. `key` - מפתח המאפיין (סטרניג או symbol) שממנו קוראים
1. `receiver` - האובייקט עליו התרחש הפעולה (בדר"כ הפרוקסי)

ארגומנטים אלה משקפים את הארגומנטים של מלכודת ה `set`, עם הבדל בולט אחד. אין `value` ארגיומנט כאן בגלל שמלכודת `get` לא כותבת ערכים. המתודה `Reflect.get()` מקבלת שלושה ארגומנטים כמו מלכודת ה `get`  ומחזירה את ערך ברירת המחדל של המאפיין.

אתה יכול להשתמש במלכודת `get` וב `Reflect.get()` לזרוק שגיאה כאשר מאםיין אינו נמצא על המטרה, כדלקמן:

</div>

```js
let proxy = new Proxy({}, {
        get(trapTarget, key, receiver) {
            if (!(key in receiver)) {
                throw new TypeError("Property " + key + " doesn't exist.");
            }

            return Reflect.get(trapTarget, key, receiver);
        }
    });

// הוספת מאפיין עדיין עובד
proxy.name = "proxy";
console.log(proxy.name);            // "proxy"

// מאפיין לא קיים יזרוק שגיאה
console.log(proxy.nme);             // throws error
```

<div dir="rtl">

בדוגמה בזו , מלכודת `get` מיירטת פעולת קרחאת מאפיין. האופרטור `in` משמש כדי לקבוע אם המאפיין כבר קיים ב- `receiver`. ה `receiver` משתמש עם  `in` במקום `trapTarget` במקרה ש `receiver` הוא פרוקסי עם מלכודת `has` , נושא שנחסה בחלק הבא. שימוש ב `trapTarget` במקרה הזה היה מפריע למלכודת `has` ועלול להעניק לך תוצאה שגויה.שגיאה נזרקת אם המאפיין אינו קיים,  אחרת משתמשים בהתנהגות ברירת המחדל.

קוד זה מאפשר מאפיינים חדשים כמו `proxy.name` להתווסף, להיות מורשי כתיבה, ולקרוא מהם בלי שום בעיה.השורה האחרונה מכילה שגיאת דפוס: `proxy.nme` צריך  להיות `proxy.name` במקום. זה יזרוק שגיאה בגלל ש `nme` לא קיים כמאפיין.

## הסתרת קיום מאפיין באמצעות מלכודת `has` 

ה `in` אופרטור בודק אם מאפיין קיים על אובייקט ומחזיר `true` אם קיים או קיים במאפייני הפרוטוטייפ המתאימים לשם או ל symbol. לדוגמא:

</div>


```js
let target = {
    value: 42;
}

console.log("value" in target);     // true
console.log("toString" in target);  // true
```

<div dir="rtl">

שניהם - גם  `value` וגם `toString` קיימים על  `object`, לכן בשני המקרים `in` האופרטור יחזיר `true`. המאפיין `value` זה מאפיין של האובייקט בזמן ש `toString` הוא מאפיין מהפרוטוטייפ (יורש `Object`). פרוקסי מאפשרים לך ליירט את הפעולה הזו ולהחזיר ערכים שונים עבור `in` עם מלכודת  `has`.

מלכודת `has` נקראית בזמן שאופרטור `in` בשימוש. כאשר נקרא, שני ארגונטים מועברים למלכודת `has`:

1. `trapTarget` - האובייקט ממנו קוראים את המאפיין (פרוקסי target)
1. `key` - מפתח המאפיין (סטרניג או symbol) שממנו קוראים

המתודה `Reflect.has()` מקבלת את אותם ארגומנטים ומחזיר את תגובת ברירת המחדל עבור `in` אופרטור. בשימוש במלכודת `has` וב `Reflect.has()` מאפשר לשנות את ההתנהגות של `in` עבור מאפיינים מסוימים תוך בזמן שיש לך גיבוי להתנהגות ברירת המחדך עבור אחרים. לדוגמה,נניח שאתה רק רוצה להסתיר את המאפיין `value` . אתה יכול לעשות את זה כך:

</div>

```js
let target = {
    name: "target",
    value: 42
};

let proxy = new Proxy(target, {
    has(trapTarget, key) {

        if (key === "value") {
            return false;
        } else {
            return Reflect.has(trapTarget, key);
        }
    }
});


console.log("value" in proxy);      // false
console.log("name" in proxy);       // true
console.log("toString" in proxy);   // true
```

<div dir="rtl">

מלכודת `has` עבור `proxy` בודק אם `key` הוא `"value"` מחזיר `false` אם כן. אחרת, התנהגות ברירת המחדל משמשת בקריאה אל המתודה `Reflect.has()` . כתוצאה, האופרטור `in` מחזיר `false` עבור המאפיין `value` למרות שהמאפיין `value` קיים על האובייקט. המאפיינים האחרים, `name` ו `toString`, מחזירים `true` כאשר משתמשים באופרטור `in`.

## מניעת מחיקת מאפיין ע"י מלכודת `deleteProperty` Trap

האופרטור `delete` מוחק מאפיין מתוך והאבייקט ומחזיר `true` כאשר מצליח ו `false` כאשר נכשל. במצב strict , `delete` יזרוק שגיאה כאשר ינסה למחוק מאפיין שלא הוגדר; במצב לא strict, `delete` יחזיר פשוט `false`. להלן דוגמא:

</div>

```js
let target = {
    name: "target",
    value: 42
};

Object.defineProperty(target, "name", { configurable: false });

console.log("value" in target);     // true

let result1 = delete target.value;
console.log(result1);               // true

console.log("value" in target);     // false

// הערה: השורה הבאה תזרוק שגיאה ב strict mode
let result2 = delete target.name;
console.log(result2);               // false

console.log("name" in target);      // true
```

<div dir="rtl">

המאפיין `value` נחמק ע"י שימוש באופרטור `delete` וכתוצאה ה `in` אופרטור מחזיר `false` בקריאה השלישית של `console.log()`. המאפיין שלא מוגדר `name` לא יכול להמחק לכן האופרטור `delete` פשוט מחזיר `false` (אם הקוד רץ ב strict הוא יזרוק שגיאה במקום). אתה יכול להתערב בהתנהגות הזו ש"י שימוש במלכודת `deleteProperty` בתוך הפרוקסי.

המלכודת `deleteProperty` נקראית כל פעם שהאופרטור `delete` הוא בשימוש על מאפיין באובייקט. המלכודת מעבירה שני ארגומנטים:

1. `trapTarget` - האובייקט ממנו מוחקים את המאפיין (פרוקסי target)
1. `key` - מפתח המאפיין (סטרניג או symbol) שאותו מוחקים

המתודה `Reflect.deleteProperty()` מספקת את היישום הדיפולטי של  המלכודת `deleteProperty` ומקבלת את אותם שני ארגומנטים. אתה יכול לשלב את `Reflect.deleteProperty()` ואת במלכודת `deleteProperty` לשנות את התנהגות האופרטור `delete` . לדוגמא,אתה יכול לוודא שהמאפיין `value` לא יוכל להמחק:

</div>

```js
let target = {
    name: "target",
    value: 42
};

let proxy = new Proxy(target, {
    deleteProperty(trapTarget, key) {

        if (key === "value") {
            return false;
        } else {
            return Reflect.deleteProperty(trapTarget, key);
        }
    }
});

// נסיון למחוק proxy.value

console.log("value" in proxy);      // true

let result1 = delete proxy.value;
console.log(result1);               // false

console.log("value" in proxy);      // true

// נסיון למחוק proxy.value

console.log("name" in proxy);       // true

let result2 = delete proxy.name;
console.log(result2);               // true

console.log("name" in proxy);       // false
```

<div dir="rtl">

הקוד נורא דומה לדומת המלכודת `has` המלכודת `deleteProperty` בודקת אם ה `key` הוא `"value"` ומחזירה `false` אם כן. אחרת, ההתנהגות הדיפולטיבית נקראית ע"י המתודה `Reflect.deleteProperty()` . המאפיין `value` לא יכול להמחק דרך `proxy` בגלל שהמאפיין ממולכד, אבל המאפיין `name` נחמק כמו שמצופה. גישה זו שימושית במיוחד כשרוצים להגן על מאפיינים מהלמחק ב strict mode בלי לזרוק שגיאה.

## מלכודת פרוקסי לאב-טיפוס (Prototype)

פרק ארבע הציג את המתודה `Object.setPrototypeOf()` ש ECMAScript 6 כדי להשלים רת המתודה של ECMAScript 5 `Object.getPrototypeOf()` . פרוקסי מאפשר לך ליירט את הביצוע של שני המתודות דרך המלכודות `setPrototypeOf` ו `getPrototypeOf` . בשני המקרים, המתודות על האובייקט `Object` קורא למלכודת של השם המקביל בפרוקסי, ומאפשר לך לשנות את התנהגות המתודות.

מאחר ויש שני מלכודות הקשורות לפרוטוטייפ פרוקסי,יש סט של מלכודות הקשורות לכל אחת מהמתודות. המלכודת `setPrototypeOf` מקבלת את הארגונטים הבאים:

1. `trapTarget` - האובייקט שעבורו יש להגדיר את אב הטיפוס (פרוקסי target)
1. `proto` - האובייקט לשימוש בו כאב-טיפוס

אלו אותם ארגונטים המועברים למתודות `Object.setPrototypeOf()` ו `Reflect.setPrototypeOf()` . המלכודת `getPrototypeOf` , מצד שני, מקבלת רק את הארגומנט `trapTarget`, שזה הארגומנט שמועבר למטודות `Object.getPrototypeOf()` ו `Reflect.getPrototypeOf()`.

### How Prototype Proxy Traps Work - איך מלכודת פרוקסי לאב-טיפוס עובדת

יש כמה מגבלות על מלכודות אלה. ראשית, מלכודת `getPrototypeOf` חייבת להחזיר אובייקט או `null`, וכל ערך החזרה אחר מביא לשגיאת זמן ריצה. בדיקת הערך החוחזר מבטיחה ש `Object.getPrototypeOf()` תמיד תחזיר את הערך הרצוי. באופן דומה, ערך ההחזר של המלכודת  `setPrototypeOf` חייב להיות `false` אם הפעולה לא מצליחה.כאשר `setPrototypeOf` מחזיר `false`, `Object.setPrototypeOf()` יזרוק שגיאה. אם `setPrototypeOf` יחזיר ערך אחר מאשר `false`, אז `Object.setPrototypeOf()` ישער שהפעולה הצליחה.

הדוגמה הבאה מסתירה את אב הטיפוס של ה- proxy על ידי החזרה תמיד של `null` וגם לא מאפשרת לשנות את האב-טיפוס:
</div>

```js
let target = {};
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return null;
    },
    setPrototypeOf(trapTarget, proto) {
        return false;
    }
});

let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);

console.log(targetProto === Object.prototype);      // true
console.log(proxyProto === Object.prototype);       // false
console.log(proxyProto);                            // null

// יצליח
Object.setPrototypeOf(target, {});

// יזרוק שגיאה
Object.setPrototypeOf(proxy, {});
```

<div dir="rtl">

קוד זה מדגיש את ההבדל בין התנהגות של `target` ו `proxy`. בזמן ש `Object.getPrototypeOf()` מחזיר את הערך של `target`, מוחזר `null` עבור `proxy` בגלל שהמלכודת `getPrototypeOf` נקראית. באופן דומה, `Object.setPrototypeOf()` מצליח כאשר משתמשים ב `target` אבל זורק שגיאה כאשר משתמשים ב `proxy` בגלל המלכודת `setPrototypeOf` .

אם ברצונך להשתמש בהתנהגות ברירת המחדל עבור שני מלכודות אלה,אתה יכול להשתמש במתודות המתאימות ב- `Reflect`. לדוגמה, קוד זה מיישם את התנהגות ברירת המחדל עבור המלכודות- `getPrototypeOf` ו `setPrototypeOf`:

</div>

```js
let target = {};
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return Reflect.getPrototypeOf(trapTarget);
    },
    setPrototypeOf(trapTarget, proto) {
        return Reflect.setPrototypeOf(trapTarget, proto);
    }
});

let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);

console.log(targetProto === Object.prototype);      // true
console.log(proxyProto === Object.prototype);       // true

//יצליח
Object.setPrototypeOf(target, {});

// גם יצליח
Object.setPrototypeOf(proxy, {});
```

<div dir="rtl">

בדוגמה זו תוכלו להשתמש גם ב `target` וגם ב `proxy` לחלופין ולקבל את אותם התוצאות בגלל שהמלכודות `getPrototypeOf` ו `setPrototypeOf` מעבירות את ההתנהגות הדיפולטיבית. אנחנו משידים זאת בדוגמא הנוכחית תודות למתודות `Reflect.getPrototypeOf()` ו `Reflect.setPrototypeOf()` ולא למתודות עם אותו שם שנמצאות על ה `Object` עם כמה הבדלים חשובים.

### למה שתי סוגי מתודות?

ההיבט המבלבל של `Reflect.getPrototypeOf()` ו `Reflect.setPrototypeOf()` זה שהם נראים מאוד דומים בצורה חשודה למתודות  `Object.getPrototypeOf()` ו `Object.setPrototypeOf()` .בזמן ששני סוגי המתודות נראים שעושים פעולות דומות , ישנם כמה הבדלים ברורים בין השניים.

נתחיל, `Object.getPrototypeOf()` ו `Object.setPrototypeOf()` הן פעולות ברמה גבוהה יותר שנוצרו לשימוש מפתחים מההתחלה. המתודות `Reflect.getPrototypeOf()` ו `Reflect.setPrototypeOf()` הן םפעולות ברמה נמוכה המעניקות למפתחים גישה למערכת הפנימית  שהיתה בעבר פנימית בלבד - האופרטורים `[[GetPrototypeOf]]` ו `[[SetPrototypeOf]]` . המתודה `Reflect.getPrototypeOf()` היא מעטפת לפעולה `[[GetPrototypeOf]]`  (עם אותו אימות קלט). המתודה `Reflect.setPrototypeOf()` ו `[[SetPrototypeOf]]` יש אותה מערכת יחסים. המתודות המתאימות על ה `Object` גם נקראות `[[GetPrototypeOf]]` ו `[[SetPrototypeOf]]` אך מבוצעת מסםר שלבים לפני הקריאה ובודקת את הערך החוזר כדי לקבוע את ההתנהגות שלו.

המתודה `Reflect.getPrototypeOf()` זורקת שגיאה אם הארגומנט הוא לא אובייקט, כאשר `Object.getPrototypeOf()` קודם יכניס את הערך לאובייקט לפני ביצוע הפעולה. אם הייתם מעבירים מספר לכל אחת מהשיטות, הייתם מקבלים תוצאה שונה:

</div>

```js
let result1 = Object.getPrototypeOf(1);
console.log(result1 === Number.prototype);  // true

// throws an error
Reflect.getPrototypeOf(1);
```

<div dir="rtl">


המתודה `Object.getPrototypeOf()` מאפשרת לך לקבל את הפרוטוטייפ של המספר `1` בגלל שקודם הוא מכפיף את הערך לאובייקט `Number` ואז מחזיר `Number.prototype`. המתודה `Reflect.getPrototypeOf()` לא מכפיפה את הערך , לכן `1` הוא לא אובייקט, וזורק שגיאה.

המתודה `Reflect.setPrototypeOf()` יש לה גם מספר שינויים מהמתודה `Object.setPrototypeOf()` . ראשית, `Reflect.setPrototypeOf()` מחזירה ערך בוליאני המציין אם הפעולה הצליחה. הערך `true` יחזור עבור הצלחה,ו  `false` יחזור עבור כישלון. אם `Object.setPrototypeOf()` נכשל, יזרוק שגיאה.

בדוגמא הראשונה תחת "מלכודת פרוקסי לאב-טיפוס" הראינו, כאשר מלכודת הפרוקסי `setPrototypeOf` מחזירה `false`, זה גורם ל `Object.setPrototypeOf()` לזרוק שגיאה. המתודה `Object.setPrototypeOf()` מחזירה את הארגומנט הראשון כערך וזה לא מתאים להתנהגות ברירת המחדל דל מלכודת הפרוקסי `setPrototypeOf` . הקוד הבא מדגים את ההבדלים הללו:

</div>

```js
let target1 = {};
let result1 = Object.setPrototypeOf(target1, {});
console.log(result1 === target1);                   // true

let target2 = {};
let result2 = Reflect.setPrototypeOf(target2, {});
console.log(result2 === target2);                   // false
console.log(result2);                               // true
```

<div dir="rtl">


בדוגמא הזו, `Object.setPrototypeOf()` מחזיר את `target1` כערך שלו, אבל `Reflect.setPrototypeOf()` מחזיר `true`. ההבדל העדין הזה חשוב מאוד. תאם תוכלו לקראות לכאורה עוד מתודות כפולות ל `Object` ו `Reflect`, אך הקפד להשתמש במתודה `Reflect` בתוך מלכודת פרוקסי.

I> שתי השיטות של המתודות יקראו למלכודת הפרוקסי  `getPrototypeOf` ו `setPrototypeOf` כאשר משתמשים בפרוקסי.

## מלכודות להרחבת אובייקט

ECMAScript 5 הוסיף אפשרות להרחבת האובייקט דרך המתודות `Object.preventExtensions()` ו `Object.isExtensible()` , ו ECMAScript 6 מאפשרת לפרוקסי ליירט את הקריאה למתודות לאובייקטים הבסיסיים באמצעות המלכודות `preventExtensions` ו `isExtensible` . שני המלכודות מקבלים ארגומנט אחד שנקרא `trapTarget` זה האובייקט עליו נקראה המתודה. המלכודת `isExtensible` חייבת להחזיר ערך בוליאני המציין אם האובייקט ניתן להרחבה ואילו המלכודת `preventExtensions`  חייבת להחזיר ערך בוליאני המציין אם הפעולה הצליחה.

יש גם את המתודות `Reflect.preventExtensions()` ו `Reflect.isExtensible()` ליישם את ההתנהגות הדיפולטיבית. שניהם מחזירים ערך בוליאני, כך שניתן להשתמש בהם ישירות במלכודות המקבילות שלהם.

### שתי דוגמאות בסיסיות

כדי לראות מלכודות הרחבה בפעולה, בקוד הבא , אשר מיישם את התנהגות ברירת המחדל עבור המלכודות `isExtensible` ו `preventExtensions` :

</div>

```js
let target = {};
let proxy = new Proxy(target, {
    isExtensible(trapTarget) {
        return Reflect.isExtensible(trapTarget);
    },
    preventExtensions(trapTarget) {
        return Reflect.preventExtensions(trapTarget);
    }
});


console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true

Object.preventExtensions(proxy);

console.log(Object.isExtensible(target));       // false
console.log(Object.isExtensible(proxy));        // false
```

<div dir="rtl">

דוגמא זו מראה ש `Object.preventExtensions()` ו `Object.isExtensible()` עוברים כראוי מ `proxy` ל `target`. אתה יכול כמובן , לשנות את ההתנהגות. לדוגמא, אם אתה לא רוצה לאפשר ל `Object.preventExtensions()` להצליח בפרוקסי שלך, אתה יכול להחזיר `false` מהמלכודת `preventExtensions` :

</div>

```js
let target = {};
let proxy = new Proxy(target, {
    isExtensible(trapTarget) {
        return Reflect.isExtensible(trapTarget);
    },
    preventExtensions(trapTarget) {
        return false
    }
});


console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true

Object.preventExtensions(proxy);

console.log(Object.isExtensible(target));       // true
console.log(Object.isExtensible(proxy));        // true
```

<div dir="rtl">

כאן בקריאה ל `Object.preventExtensions(proxy)` מתעלם ביעילות מכיוון שהמלכודת `preventExtensions` מחזירה `false`. הפעולה אינה מועברת לבסיס `target`, כך ש `Object.isExtensible()` מחזיר `true`.

### שיטות הרחבה כפולות

יתכן ששמת לב, שיש דמיון וכפילויות במתודות ב `Object` ו `Reflect`. במקרה הזה יש יותר דמיון מאשר לא. המתודות  `Object.isExtensible()` ו `Reflect.isExtensible()` הם דומים למעט כאשר מועבר ערך שהוא אינו אובייקט. במקרה הזה, `Object.isExtensible()` תמיד יחזיר `false` כאשר `Reflect.isExtensible()` יזרוק שגיאה. הנה דוגמא להתנהגות הזו:

</div>

```js
let result1 = Object.isExtensible(2);
console.log(result1);                       // false

// יזרוק שגיאה
let result2 = Reflect.isExtensible(2);
```

<div dir="rtl">

מגבלה זו דומה להבדל בין המתודות `Object.getPrototypeOf()` ו `Reflect.getPrototypeOf()` , מכיוון  שלמתודות עם פונקונאליות ברמה הנמוכה יש יותר בדיקות שגיאה מחמירות מאשר  למקבילה ברמה הגבוהה יותר.

המתודות  `Object.preventExtensions()` ו `Reflect.preventExtensions()` גם כן נורא דומות. המתודה `Object.preventExtensions()` תמיד תחזיר את הערך שהועובר אליה כארגומנט אפילו אם הערך הוא אינו אובייקט. המתודה `Reflect.preventExtensions()` , מצד שני , תזרוק שגיאה אם הארגומנט הוא אינו אובייקט; אם הארגומנט הוא אובייקט, אזי `Reflect.preventExtensions()` יחזיר `true` כאשר הפעולה מצליחה  ויחזיר `false` אם לא. לדוגמא:

</div>

```js
let result1 = Object.preventExtensions(2);
console.log(result1);                               // 2

let target = {};
let result2 = Reflect.preventExtensions(target);
console.log(result2);                               // true

// יזרוק שגיאה
let result3 = Reflect.preventExtensions(2);
```

<div dir="rtl">

כאן מועבר ל `Object.preventExtensions()` הערך `2` והוא מחזיר למרות ש`2` הוא לא אובייקט. המתודה `Reflect.preventExtensions()` מחזירה `true` כאשר אובייקט מועובר אליה וזורקת שגיאה כאשר  `2` מועבר אליה.

## מלכודת לתיאור מאפיינים

אחת התכונות החשובות ביותר של ECMAScript 5 היא היכולת להגדיר מאפיינים -property attributes בשימוש עם המתודה  `Object.defineProperty()` . בגרסאות קודמות של ג'אווה סקריפט, לא הייתה שום דרך להגדיר מאפיין , להגדיר מאפיין רק לקריאה, או בלתי נספר. כל זה אפשרי תודות למתודה `Object.defineProperty()` ,ואתה יכול לקבל מאפיין תודות למתודה `Object.getOwnPropertyDescriptor()`.

פרוקסי נותן לך אפשרות ליירט קריאות ל `Object.defineProperty()` ו `Object.getOwnPropertyDescriptor()` בשימוש המלכודות `defineProperty` ו `getOwnPropertyDescriptor` , בהתאמה. המלכודת `defineProperty` מקבלת את הארגומנטים הבאים:

1. `trapTarget` - האובייקט שיקבל את המאפיינים (המטרה של הפרוקסי)
1. `key` - מפתח המאפיין (סטרינג או symbol) לכתוב אליו
1. `descriptor` - אובייקט המתאר את המאפיין

מלכודת `defineProperty` דורשת שתחזיר `true` אם הפעולה הצליחה ו `false` אם לא. מלכודת `getOwnPropertyDescriptor` מקבלת רק `trapTarget` ו `key`, ואתה מצפה שתחזיר את המתאר. המתודות התואמות `Reflect.defineProperty()` ו `Reflect.getOwnPropertyDescriptor()` מקבלות את אותם ארגומנטים כמו מלכודת הפרוקסי המקבילה. להלן דוגמא שמיישמת את התנהגות ברירת המחדל עבור כל מלכודת:

</div>

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {
        return Reflect.defineProperty(trapTarget, key, descriptor);
    },
    getOwnPropertyDescriptor(trapTarget, key) {
        return Reflect.getOwnPropertyDescriptor(trapTarget, key);
    }
});


Object.defineProperty(proxy, "name", {
    value: "proxy"
});

console.log(proxy.name);            // "proxy"

let descriptor = Object.getOwnPropertyDescriptor(proxy, "name");

console.log(descriptor.value);      // "proxy"
```

<div dir="rtl">

הקוד פה מגדיר מאפיין `"name"` על הפרוקסי עם המתודה  `Object.defineProperty()` . לאחר מכן מאוחזר מתאר המאפיינים של אותו נכס במתודה `Object.getOwnPropertyDescriptor()` .

### חסימת Object.defineProperty()

מלכודת `defineProperty` דורשת שתחזיר עקך בוליאני כדי לציין אם הפעולה הצליחה. כאשר מוחזר `true` , `Object.defineProperty()` הצליח כרגיל; כאשר מוחזר `false` , `Object.defineProperty()` יזרוק שגיאה. אתה יכול להשתמש בפונקציונליות זו כדי להגביל את סוגי המאפיינים שהמתודה- `Object.defineProperty()` יכולה להגדיר. לדוגמה, אם אתה רוצה למנוע את הגדרת מאפייני ה symbol , אתה יכול לבדוק שהמפתח הוא מחרוזת ולהחזיר `false` , כמו כאן:

</div>

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {

        if (typeof key === "symbol") {
            return false;
        }

        return Reflect.defineProperty(trapTarget, key, descriptor);
    }
});


Object.defineProperty(proxy, "name", {
    value: "proxy"
});

console.log(proxy.name);                    // "proxy"

let nameSymbol = Symbol("name");

// יזרוק שגיאה
Object.defineProperty(proxy, nameSymbol, {
    value: "proxy"
});
```

<div dir="rtl">

מלכודת הפרוקסי `defineProperty` מחזירה `false` אם `key` הוא symbol אחרת ממשיכה עם ההתנהגות הדיפולטיבית. כאשר `Object.defineProperty()` נקרא עם  `"name"` כמפתח, המתודה תצליח בגלל שהמפתח הוא סטרינג. כאשר `Object.defineProperty()` נקראית עם  `nameSymbol`, הוא יזרוק שגיאה כי המלכודת `defineProperty` מחזירה `false`.

I> אתה יכול גם לקבל את `Object.defineProperty ()` שיכשל בשקט על ידי החזרת `true` ולא לקרוא לשיטה `Reflect.defineProperty ()`. זה ידכא את השגיאה אם לא הגדרת את המאפיין בפועל.

### הגבלת מתאר אובייקט

על מנת להבטיח התנהגות עקבית בזמן השימוש במתודות `Object.defineProperty()` ו `Object.getOwnPropertyDescriptor()` , מתאר אובייקט שמועבר למלכודת `defineProperty` מנורמלים. אובייקטים שמוחזרים מהמלכודת `getOwnPropertyDescriptor` תמיד יעברו אימות מאותה הסיבה.

לא משנה איזה אובייקט מועבר בארגומנט השלישי למתודה `Object.defineProperty()` , רק המאפיינים `enumerable`, `configurable`, `value`, `writable`, `get`, ו `set` יהיו על מתאר האובייקט המועבר למלכודת  `defineProperty` . לדוגמא:

</div>

```js
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {
        console.log(descriptor.value);              // "proxy"
        console.log(descriptor.name);               // undefined

        return Reflect.defineProperty(trapTarget, key, descriptor);
    }
});


Object.defineProperty(proxy, "name", {
    value: "proxy",
    name: "custom"
});
```

<div dir="rtl">

כאן, `Object.defineProperty()` קורא למאפיין לא סטנדרטי `name` בארגומנט השלישי. כאשר מלכודת `defineProperty` נקראית, ל `descriptor` של האובייקט אין את המאפיין `name` אבל יש לו את המאפיין `value` . זה בגלל ש `descriptor`  לא מתייחס בפועל לארגומנט השלישי שהועובר במתודה  `Object.defineProperty()` , אלא לאובייקט החדש המכיל רק את המאפיינים המותרים. המתודה `Reflect.defineProperty()` גם מתעלמת מהמאפיינים שלא מותרים במתאר.

המלכודת `getOwnPropertyDescriptor` יש מגבלה מעט שונה שדורשת שיחזיר ערך של  `null`, `undefined`, או אובייקט. אם מוחזר אובייקט, רק המאפיינים `enumerable`, `configurable`, `value`, `writable`, `get`, ו `set` מותרים כמאפיינים על האובייקט. שגיאה נזרקת אם אתה מחזיר אובייקט עם מאפיין משלו שאינו מותר, כמו שהקוד מראה:

</div>

```js
let proxy = new Proxy({}, {
    getOwnPropertyDescriptor(trapTarget, key) {
        return {
            name: "proxy"
        };
    }
});

// יזרוק שגיאה
let descriptor = Object.getOwnPropertyDescriptor(proxy, "name");
```

<div dir="rtl">

המאפיין `name` לא מאושר כמאפיין על המתאר, כך ש `Object.getOwnPropertyDescriptor()` נקרא, ה `getOwnPropertyDescriptor` ערך החזרה מפעיל שגיאה. הגבלה זו מבטיחה שהערך שיוחזר על ידי `Object.getOwnPropertyDescriptor()`תמיד יש מבנה אמין ללא קשר לשימוש בפרוקסי.

### Duplicate Descriptor Methods

Once again, ECMAScript 6 has some confusingly similar methods, as the `Object.defineProperty()` and `Object.getOwnPropertyDescriptor()` methods appear to do the same thing as the `Reflect.defineProperty()` and `Reflect.getOwnPropertyDescriptor()` methods, respectively. Like other method pairs discussed earlier in this chapter, these have some subtle but important differences.

#### defineProperty() Methods

The `Object.defineProperty()` and `Reflect.defineProperty()` methods are exactly the same except for their return values. The `Object.defineProperty()` method returns the first argument, while `Reflect.defineProperty()` returns `true` if the operation succeeded and `false` if not. For example:


```js
let target = {};

let result1 = Object.defineProperty(target, "name", { value: "target "});

console.log(target === result1);        // true

let result2 = Reflect.defineProperty(target, "name", { value: "reflect" });

console.log(result2);                   // true
```

When `Object.defineProperty()` is called on `target`, the return value is `target`. When `Reflect.defineProperty()` is called on `target`, the return value is `true`, indicating that the operation succeeded. Since the `defineProperty` proxy trap requires a boolean value to be returned, it's better to use `Reflect.defineProperty()` to implement the default behavior when necessary.

#### getOwnPropertyDescriptor() Methods

The `Object.getOwnPropertyDescriptor()` method coerces its first argument into an object when a primitive value is passed and then continues the operation. On the other hand, the `Reflect.getOwnPropertyDescriptor()` method throws an error if the first argument is a primitive value. Here's an example showing both:

```js
let descriptor1 = Object.getOwnPropertyDescriptor(2, "name");
console.log(descriptor1);       // undefined

// throws an error
let descriptor2 = Reflect.getOwnPropertyDescriptor(2, "name");
```

The `Object.getOwnPropertyDescriptor()` method returns `undefined` because it coerces `2` into an object, and that object has no `name` property. This is the standard behavior of the method when a property with the given name isn't found on an object. When `Reflect.getOwnPropertyDescriptor()` is called, however, an error is thrown immediately because that method doesn't accept primitive values for the first argument.

## The `ownKeys` Trap

The `ownKeys` proxy trap intercepts the internal method `[[OwnPropertyKeys]]` and allows you to override that behavior by returning an array of values. This array is used in four methods: the `Object.keys()` method, the `Object.getOwnPropertyNames()` method, the `Object.getOwnPropertySymbols()` method, and the `Object.assign()` method. (The `Object.assign()` method uses the array to determine which properties to copy.)

The default behavior for the `ownKeys` trap is implemented by the `Reflect.ownKeys()` method and returns an array of all own property keys, including both strings and symbols. The `Object.getOwnProperyNames()` method and the `Object.keys()` method filter symbols out of the array and returns the result while `Object.getOwnPropertySymbols()` filters the strings out of the array and returns the result. The `Object.assign()` method uses the array with both strings and symbols.

The `ownKeys` trap receives a single argument, the target, and must always return an array or array-like object; otherwise, an error is thrown. You can use the `ownKeys` trap to, for example, filter out certain property keys that you don't want used when the `Object.keys()`, the `Object.getOwnPropertyNames()` method, the `Object.getOwnPropertySymbols()` method, or the `Object.assign()` method is used. Suppose you don't want to include any property names that begin with an underscore character, a common notation in JavaScript indicating that a field is private. You can use the `ownKeys` trap to filter out those keys as follows:

```js
let proxy = new Proxy({}, {
    ownKeys(trapTarget) {
        return Reflect.ownKeys(trapTarget).filter(key => {
            return typeof key !== "string" || key[0] !== "_";
        });
    }
});

let nameSymbol = Symbol("name");

proxy.name = "proxy";
proxy._name = "private";
proxy[nameSymbol] = "symbol";

let names = Object.getOwnPropertyNames(proxy),
    keys = Object.keys(proxy);
    symbols = Object.getOwnPropertySymbols(proxy);

console.log(names.length);      // 1
console.log(names[0]);          // "name"

console.log(keys.length);      // 1
console.log(keys[0]);          // "name"

console.log(symbols.length);    // 1
console.log(symbols[0]);        // "Symbol(name)"
```

This example uses an `ownKeys` trap that first calls `Reflect.ownKeys()` to get the default list of keys for the target. Then, the `filter()` method is used to filter out keys that are strings and begin with an underscore character. Then, three properties are added to the `proxy` object: `name`, `_name`, and `nameSymbol`. When `Object.getOwnPropertyNames()` and `Object.keys()` is called on `proxy`, only the `name` property is returned. Similarly, only `nameSymbol` is returned when `Object.getOwnPropertySymbols()` is called on `proxy`. The `_name` property doesn't appear in either result because it is filtered out.

I> The `ownKeys` trap also affects the `for-in` loop, which calls the trap to determine which keys to use inside of the loop.

## Function Proxies with the `apply` and `construct` Traps

Of all the proxy traps, only `apply` and `construct` require the proxy target to be a function. Recall from Chapter 3 that functions have two internal methods called `[[Call]]` and `[[Construct]]` that are executed when a function is called without and with the `new` operator, respectively. The `apply` and `construct` traps correspond to and let you override those internal methods. When a function is called without `new`, the `apply` trap receives, and `Reflect.apply()` expects, the following arguments:

1. `trapTarget` - the function being executed (the proxy's target)
1. `thisArg` - the value of `this` inside of the function during the call
1. `argumentsList` - an array of arguments passed to the function

The `construct` trap, which is called when the function is executed using `new`, receives the following arguments:

1. `trapTarget` - the function being executed (the proxy's target)
1. `argumentsList` - an array of arguments passed to the function

The `Reflect.construct()` method also accepts these two arguments and has an optional third argument called `newTarget`. When given, the `newTarget` argument specifies the value of `new.target` inside of the function.

Together, the `apply` and `construct` traps completely control the behavior of any proxy target function. To mimic the default behavior of a function, you can do this:

```js
let target = function() { return 42 },
    proxy = new Proxy(target, {
        apply: function(trapTarget, thisArg, argumentList) {
            return Reflect.apply(trapTarget, thisArg, argumentList);
        },
        construct: function(trapTarget, argumentList) {
            return Reflect.construct(trapTarget, argumentList);
        }
    });

// a proxy with a function as its target looks like a function
console.log(typeof proxy);                  // "function"

console.log(proxy());                       // 42

var instance = new proxy();
console.log(instance instanceof proxy);     // true
console.log(instance instanceof target);    // true
```

This example has a function that returns the number 42. The proxy for that function uses the `apply` and `construct` traps to delegate those behaviors to the `Reflect.apply()` and `Reflect.construct()` methods, respectively. The end result is that the proxy function works exactly like the target function, including identifying itself as a function when `typeof` is used. The proxy is called without `new` to return 42 and then is called with `new` to create an object called `instance`. The `instance` object is considered an instance of both `proxy` and `target` because `instanceof` uses the prototype chain to determine this information. Prototype chain lookup is not affected by this proxy, which is why `proxy` and `target` appear to have the same prototype to the JavaScript engine.

### Validating Function Parameters

The `apply` and `construct` traps open up a lot of possibilities for altering the way a function is executed. For instance, suppose you want to validate that all arguments are of a specific type. You can check the arguments in the `apply` trap:

```js
// adds together all arguments
function sum(...values) {
    return values.reduce((previous, current) => previous + current, 0);
}

let sumProxy = new Proxy(sum, {
        apply: function(trapTarget, thisArg, argumentList) {

            argumentList.forEach((arg) => {
                if (typeof arg !== "number") {
                    throw new TypeError("All arguments must be numbers.");
                }
            });

            return Reflect.apply(trapTarget, thisArg, argumentList);
        },
        construct: function(trapTarget, argumentList) {
            throw new TypeError("This function can't be called with new.");
        }
    });

console.log(sumProxy(1, 2, 3, 4));          // 10

// throws error
console.log(sumProxy(1, "2", 3, 4));

// also throws error
let result = new sumProxy();
```

This example uses the `apply` trap to ensure that all arguments are numbers. The `sum()` function adds up all of the arguments that are passed. If a non-number value is passed, the function will still attempt the operation, which can cause unexpected results. By wrapping `sum()` inside the `sumProxy()` proxy, this code intercepts function calls and ensures that each argument is a number before allowing the call to proceed. To be safe, the code also uses the `construct` trap to ensure that the function can't be called with `new`.

You can also do the opposite, ensuring that a function must be called with `new` and validating its arguments to be numbers:

```js
function Numbers(...values) {
    this.values = values;
}

let NumbersProxy = new Proxy(Numbers, {

        apply: function(trapTarget, thisArg, argumentList) {
            throw new TypeError("This function must be called with new.");
        },

        construct: function(trapTarget, argumentList) {
            argumentList.forEach((arg) => {
                if (typeof arg !== "number") {
                    throw new TypeError("All arguments must be numbers.");
                }
            });

            return Reflect.construct(trapTarget, argumentList);
        }
    });

let instance = new NumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]

// throws error
NumbersProxy(1, 2, 3, 4);
```

Here, the `apply` trap throws an error while the `construct` trap uses the `Reflect.construct()` method to validate input and return a new instance. Of course, you can accomplish the same thing without proxies using `new.target` instead.

### Calling Constructors Without new

Chapter 3 introduced the `new.target` metaproperty. To review, `new.target` is a reference to the function on which `new` is called, meaning that you can tell if a function was called using `new` or not by checking the value of `new.target` like this:

```js
function Numbers(...values) {

    if (typeof new.target === "undefined") {
        throw new TypeError("This function must be called with new.");
    }

    this.values = values;
}

let instance = new Numbers(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]

// throws error
Numbers(1, 2, 3, 4);
```

This example throws an error when `Numbers` is called without using `new`, which is similar to the example in the "Validating Function Parameters" section but doesn't use a proxy. Writing code like this is much simpler than using a proxy and is preferable if your only goal is to prevent calling the function without `new`. But sometimes you aren't in control of the function whose behavior needs to be modified. In that case, using a proxy makes sense.

Suppose the `Numbers` function is defined in code you can't modify. You know that the code relies on `new.target` and want to avoid that check while still calling the function. The behavior when using `new` is already set, so you can just use the `apply` trap:

```js
function Numbers(...values) {

    if (typeof new.target === "undefined") {
        throw new TypeError("This function must be called with new.");
    }

    this.values = values;
}


let NumbersProxy = new Proxy(Numbers, {
        apply: function(trapTarget, thisArg, argumentsList) {
            return Reflect.construct(trapTarget, argumentsList);
        }
    });


let instance = NumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]
```

The `NumbersProxy` function allows you to call `Numbers` without using `new` and have it behave as if `new` were used. To do so, the `apply` trap calls `Reflect.construct()` with the arguments passed into `apply`. The `new.target` inside of `Numbers` is equal to `Numbers` itself, and no error is thrown. While this is a simple example of modifying `new.target`, you can also do so more directly.

### Overriding Abstract Base Class Constructors

You can go one step further and specify the third argument to `Reflect.construct()` as the specific value to assign to `new.target`. This is useful when a function is checking `new.target` against a known value, such as when creating an abstract base class constructor (discussed in Chapter 9). In an abstract base class constructor, `new.target` is expected to be something other than the class constructor itself, as in this example:

```js
class AbstractNumbers {

    constructor(...values) {
        if (new.target === AbstractNumbers) {
            throw new TypeError("This function must be inherited from.");
        }

        this.values = values;
    }
}

class Numbers extends AbstractNumbers {}

let instance = new Numbers(1, 2, 3, 4);
console.log(instance.values);           // [1,2,3,4]

// throws error
new AbstractNumbers(1, 2, 3, 4);
```

When `new AbstractNumbers()` is called, `new.target` is equal to `AbstractNumbers` and an error is thrown. Calling `new Numbers()` still works because `new.target` is equal to `Numbers`. You can bypass this restriction by manually assigning `new.target` with a proxy:

```js
class AbstractNumbers {

    constructor(...values) {
        if (new.target === AbstractNumbers) {
            throw new TypeError("This function must be inherited from.");
        }

        this.values = values;
    }
}

let AbstractNumbersProxy = new Proxy(AbstractNumbers, {
        construct: function(trapTarget, argumentList) {
            return Reflect.construct(trapTarget, argumentList, function() {});
        }
    });


let instance = new AbstractNumbersProxy(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]
```

The `AbstractNumbersProxy` uses the `construct` trap to intercept the call to the `new AbstractNumbersProxy()` method. Then, the `Reflect.construct()` method is called with arguments from the trap and adds an empty function as the third argument. That empty function is used as the value of `new.target` inside of the constructor. Because `new.target` is not equal to `AbstractNumbers`, no error is thrown and the constructor executes completely.

### Callable Class Constructors

Chapter 9 explained that class constructors must always be called with `new`. That happens because the internal `[[Call]]` method for class constructors is specified to throw an error. But proxies can intercept calls to the `[[Call]]` method, meaning you can effectively create callable class constructors by using a proxy. For instance, if you want a class constructor to work without using `new`, you can use the `apply` trap to create a new instance. Here's some sample code:

```js
class Person {
    constructor(name) {
        this.name = name;
    }
}

let PersonProxy = new Proxy(Person, {
        apply: function(trapTarget, thisArg, argumentList) {
            return new trapTarget(...argumentList);
        }
    });


let me = PersonProxy("Nicholas");
console.log(me.name);                   // "Nicholas"
console.log(me instanceof Person);      // true
console.log(me instanceof PersonProxy); // true
```

The `PersonProxy` object is a proxy of the `Person` class constructor. Class constructors are just functions, so they behave like functions when used in proxies. The `apply` trap overrides the default behavior and instead returns a new instance of `trapTarget` that's equal to `Person`. (I used `trapTarget` in this example to show that you don't need to manually specify the class.) The `argumentList` is passed to `trapTarget` using the spread operator to pass each argument separately. Calling `PersonProxy()` without using `new` returns an instance of `Person`; if you attempt to call `Person()` without `new`, the constructor will still throw an error. Creating callable class constructors is something that is only possible using proxies.

## Revocable Proxies

Normally, a proxy can't be unbound from its target once the proxy has been created. All of the examples to this point in this chapter have used nonrevocable proxies. But there may be situations when you want to revoke a proxy so that it can no longer be used. You'll find it most helpful to revoke proxies when you want to provide an object through an API for security purposes and maintain the ability to cut off access to some functionality at any point in time.

You can create revocable proxies with the `Proxy.revocable()` method, which takes the same arguments as the `Proxy` constructor--a target object and the proxy handler. The return value is an object with the following properties:

1. `proxy` - the proxy object that can be revoked
1. `revoke` - the function to call to revoke the proxy

When the `revoke()` function is called, no further operations can be performed through the `proxy`. Any attempt to interact with the proxy object in a way that would trigger a proxy trap throws an error. For example:

```js
let target = {
    name: "target"
};

let { proxy, revoke } = Proxy.revocable(target, {});

console.log(proxy.name);        // "target"

revoke();

// throws error
console.log(proxy.name);
```

This example creates a revocable proxy. It uses destructuring to assign the `proxy` and `revoke` variables to the properties of the same name on the object returned by the `Proxy.revocable()` method. After that, the `proxy` object can be used just like a nonrevocable proxy object, so `proxy.name` returns `"target"` because it passes through to `target.name`. Once the `revoke()` function is called, however, `proxy` no longer functions. Attempting to access `proxy.name` throws an error, as will any other operation that would trigger a trap on `proxy`.

## Solving the Array Problem

At the beginning of this chapter, I explained how developers couldn't mimic the behavior of an array accurately in JavaScript prior to ECMAScript 6. Proxies and the reflection API allow you to create an object that behaves in the same manner as the built-in `Array` type when properties are added and removed. To refresh your memory, here's an example showing the behavior that proxies help to mimick:

```js
let colors = ["red", "green", "blue"];

console.log(colors.length);         // 3

colors[3] = "black";

console.log(colors.length);         // 4
console.log(colors[3]);             // "black"

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green"
```

There are two particularly important behaviors to notice in this example:

1. The `length` property is increased to 4 when `colors[3]` is assigned a value.
1. The last two items in the array are deleted when the `length` property is set to 2.

These two behaviors are the only ones that need to be mimicked to accurately recreate how built-in arrays work. The next few sections describe how to make an object that correctly mimics them.

### Detecting Array Indices

Keep in mind that assigning to an integer property key is a special case for arrays, as those are treated differently from non-integer keys. The ECMAScript 6 specification gives these instructions on how to determine if a property key is an array index:

> A String property name `P` is an array index if and only if `ToString(ToUint32(P))` is equal to `P` and `ToUint32(P)` is not equal to 2^32^-1.

This operation can be implemented in JavaScript as follows:

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}
```

The `toUint32()` function converts a given value into an unsigned 32-bit integer using an algorithm described in the specification. The `isArrayIndex()` function first converts the key into a uint32 and then performs the comparisons to determine if the key is an array index or not. With these utility functions available, you can start to implement an object that will mimic a built-in array.

### Increasing length when Adding New Elements

You might have noticed that both array behaviors I described rely on the assignment of a property. That means you really only need to use the `set` proxy trap to accomplish both behaviors. To get started, here's an example that implements the first of the two behaviors by incrementing the `length` property when an array index larger than `length - 1` is used:

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

function createMyArray(length=0) {
    return new Proxy({ length }, {
        set(trapTarget, key, value) {

            let currentLength = Reflect.get(trapTarget, "length");

            // the special case
            if (isArrayIndex(key)) {
                let numericKey = Number(key);

                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, "length", numericKey + 1);
                }
            }

            // always do this regardless of key type
            return Reflect.set(trapTarget, key, value);
        }
    });
}

let colors = createMyArray(3);
console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";

console.log(colors.length);         // 3

colors[3] = "black";

console.log(colors.length);         // 4
console.log(colors[3]);             // "black"
```

This example uses the `set` proxy trap to intercept the setting of an array index. If the key is an array index, then it is converted into a number because keys are always passed as strings. Next, if that numeric value is greater than or equal to the current `length` property, then the `length` property is updated to be one more than the numeric key (setting an item in position 3 means the `length` must be 4). After that, the default behavior for setting a property is used via `Reflect.set()`, since you do want the property to receive the value as specified.

The initial custom array is created by calling `createMyArray()` with a `length` of 3 and the values for those three items are added immediately afterward. The `length` property correctly remains 3 until the value `"black"` is assigned to position 3. At that point, `length` is set to 4.

With the first behavior working, it's time to move on to the second.

### Deleting Elements on Reducing length

The first array behavior to mimic is used only when an array index is greater than or equal to the `length` property. The second behavior does the opposite and removes array items when the `length` property is set to a smaller value than it previously contained. That involves not only changing the `length` property, but also deleting all items that might otherwise exist. For instance, if an array with a `length` of 4 has `length` set to 2, the items in positions 2 and 3 are deleted. You can accomplish this inside the `set` proxy trap alongside the first behavior. Here's the previous example again, with an updated `createMyArray` method:

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

function createMyArray(length=0) {
    return new Proxy({ length }, {
        set(trapTarget, key, value) {

            let currentLength = Reflect.get(trapTarget, "length");

            // the special case
            if (isArrayIndex(key)) {
                let numericKey = Number(key);

                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, "length", numericKey + 1);
                }
            } else if (key === "length") {

                if (value < currentLength) {
                    for (let index = currentLength - 1; index >= value; index--) {
                        Reflect.deleteProperty(trapTarget, index);
                    }
                }

            }

            // always do this regardless of key type
            return Reflect.set(trapTarget, key, value);
        }
    });
}

let colors = createMyArray(3);
console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";
colors[3] = "black";

console.log(colors.length);         // 4

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green"
console.log(colors[0]);             // "red"
```

The `set` proxy trap in this code checks to see if `key` is `"length"` in order to adjust the rest of the object correctly. When that happens, the current length is first retrieved using `Reflect.get()` and compared against the new value. If the new value is less than the current length, then a `for` loop deletes all properties on the target that should no longer be available. The `for` loop goes backward from the current array length (`currentLength`) and deletes each property until it reaches the new array length (`value`).

This example adds four colors to `colors` and then sets the `length` property to 2. That effectively removes the items in positions 2 and 3, so they now return `undefined` when you attempt to access them. The `length` property is correctly set to 2 and the items in positions 0 and 1 are still accessible.

With both behaviors implemented, you can easily create an object that mimics the behavior of built-in arrays. But doing so with a function isn't as desirable as creating a class to encapsulate this behavior, so the next step is to implement this functionality as a class.

### Implementing the MyArray Class

The simplest way to create a class that uses a proxy is to define the class as usual and then return a proxy from the constructor. That way, the object returned when a class is instantiated will be the proxy instead of the instance. (The instance is the value of `this` inside the constructor.) The instance becomes the target of the proxy and the proxy is returned as if it were the instance. The instance will be completely private and you won't be able to access it directly, though you'll be able to access it indirectly through the proxy.

Here's a simple example of returning a proxy from a class constructor:

```js
class Thing {
    constructor() {
        return new Proxy(this, {});
    }
}

let myThing = new Thing();
console.log(myThing instanceof Thing);      // true
```

In this example, the class `Thing` returns a proxy from its constructor. The proxy target is `this` and the proxy is returned from the constructor. That means `myThing` is actually a proxy even though it was created by calling the `Thing` constructor. Because proxies pass through their behavior to their targets, `myThing` is still considered an instance of `Thing`, making the proxy completely transparent to anyone using the `Thing` class.

With that in mind, creating a custom array class using a proxy in relatively straightforward. The code is mostly the same as the code in the "Deleting Elements on Reducing Length" section. The same proxy code is used, but this time, it's inside a class constructor. Here's the complete example:

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

class MyArray {
    constructor(length=0) {
        this.length = length;

        return new Proxy(this, {
            set(trapTarget, key, value) {

                let currentLength = Reflect.get(trapTarget, "length");

                // the special case
                if (isArrayIndex(key)) {
                    let numericKey = Number(key);

                    if (numericKey >= currentLength) {
                        Reflect.set(trapTarget, "length", numericKey + 1);
                    }
                } else if (key === "length") {

                    if (value < currentLength) {
                        for (let index = currentLength - 1; index >= value; index--) {
                            Reflect.deleteProperty(trapTarget, index);
                        }
                    }

                }

                // always do this regardless of key type
                return Reflect.set(trapTarget, key, value);
            }
        });

    }
}


let colors = new MyArray(3);
console.log(colors instanceof MyArray);     // true

console.log(colors.length);         // 3

colors[0] = "red";
colors[1] = "green";
colors[2] = "blue";
colors[3] = "black";

console.log(colors.length);         // 4

colors.length = 2;

console.log(colors.length);         // 2
console.log(colors[3]);             // undefined
console.log(colors[2]);             // undefined
console.log(colors[1]);             // "green"
console.log(colors[0]);             // "red"
```

This code creates a `MyArray` class that returns a proxy from its constructor. The `length` property is added in the constructor (initialized to either the value that is passed in or to a default value of 0) and then a proxy is created and returned. This gives the `colors` variable the appearance of being just an instance of `MyArray` and implements both of the key array behaviors.

Although returning a proxy from a class constructor is easy, it does mean that a new proxy is created for every instance. There is, however, a way to have all instances share one proxy: you can use the proxy as a prototype.

## Using a Proxy as a Prototype

Proxies can be used as prototypes, but doing so is a bit more involved than the previous examples in this chapter. When a proxy is a prototype, the proxy traps are only called when the default operation would normally continue on to the prototype, which does limit a proxy's capabilities as a prototype. Consider this example:

```js
let target = {};
let newTarget = Object.create(new Proxy(target, {

    // never called
    defineProperty(trapTarget, name, descriptor) {

        // would cause an error if called
        return false;
    }
}));

Object.defineProperty(newTarget, "name", {
    value: "newTarget"
});

console.log(newTarget.name);                    // "newTarget"
console.log(newTarget.hasOwnProperty("name"));  // true
```

The `newTarget` object is created with a proxy as the prototype. Making `target` the proxy target effectively makes `target` the prototype of `newTarget` because the proxy is transparent. Now, proxy traps will only be called if an operation on `newTarget` would pass the operation through to happen on `target`.

The `Object.defineProperty()` method is called on `newTarget` to create an own property called `name`. Defining a property on an object isn't an operation that normally continues to the object's prototype, so the `defineProperty` trap on the proxy is never called and the `name` property is added to `newTarget` as an own property.

While proxies are severely limited when used as prototypes, there are a few traps that are still useful.

### Using the `get` Trap on a Prototype

When the internal `[[Get]]` method is called to read a property, the operation looks for own properties first. If an own property with the given name isn't found, then the operation continues to the prototype and looks for a property there. The process continues until there are no further prototypes to check.

Thanks to that process, if you set up a `get` proxy trap, the trap will be called on a prototype whenever an own property of the given name doesn't exist. You can use the `get` trap to prevent unexpected behavior when accessing properties that you can't guarantee will exist. Just create an object that throws an error whenever you try to access a property that doesn't exist:

```js
let target = {};
let thing = Object.create(new Proxy(target, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
}));

thing.name = "thing";

console.log(thing.name);        // "thing"

// throw an error
let unknown = thing.unknown;
```

In this code, the `thing` object is created with a proxy as its prototype. The `get` trap throws an error when called to indicate that the given key doesn't exist on the `thing` object. When `thing.name` is read, the operation never calls the `get` trap on the prototype because the property exists on `thing`. The `get` trap is called only when the `thing.unknown` property, which doesn't exist, is accessed.

When the last line executes, `unknown` isn't an own property of `thing`, so the operation continues to the prototype. The `get` trap then throws an error. This type of behavior can be very useful in JavaScript, where unknown properties silently return `undefined` instead of throwing an error (as happens in other languages).

It's important to understand that in this example, `trapTarget` and `receiver` are different objects. When a proxy is used as a prototype, the `trapTarget` is the prototype object itself while the `receiver` is the instance object. In this case, that means `trapTarget` is equal to `target` and `receiver` is equal to `thing`. That allows you access both to the original target of the proxy and the object on which the operation is meant to take place.

### Using the `set` Trap on a Prototype

The internal `[[Set]]` method also checks for own properties and then continues to the prototype if needed. When you assign a value to an object property, the value is assigned to the own property with the same name if it exists. If no own property with the given name exists, then the operation continues to the prototype. The tricky part is that even though the assignment operation continues to the prototype, assigning a value to that property will create a property on the instance (not the prototype) by default, regardless of whether a property of that name exists on the prototype.

To get a better idea of when the `set` trap will be called on a prototype and when it won't, consider the following example showing the default behavior:

```js
let target = {};
let thing = Object.create(new Proxy(target, {
    set(trapTarget, key, value, receiver) {
        return Reflect.set(trapTarget, key, value, receiver);
    }
}));

console.log(thing.hasOwnProperty("name"));      // false

// triggers the `set` proxy trap
thing.name = "thing";

console.log(thing.name);                        // "thing"
console.log(thing.hasOwnProperty("name"));      // true

// does not trigger the `set` proxy trap
thing.name = "boo";

console.log(thing.name);                        // "boo"
```

In this example, `target` starts with no own properties. The `thing` object has a proxy as its prototype that defines a `set` trap to catch the creation of any new properties. When `thing.name` is assigned `"thing"` as its value, the `set` proxy trap is called because `thing` doesn't have an own property called `name`. Inside the `set` trap, `trapTarget` is equal to `target` and `receiver` is equal to `thing`. The operation should ultimately create a new property on `thing`, and fortunately `Reflect.set()` implements this default behavior for you if you pass in `receiver` as the fourth argument.

Once the `name` property is created on `thing`, setting `thing.name` to a different value will no longer call the `set` proxy trap. At that point, `name` is an own property so the `[[Set]]` operation never continues on to the prototype.

### Using the `has` Trap on a Prototype

Recall that the `has` trap intercepts the use of the `in` operator on objects. The `in` operator searches first for an object's own property with the given name. If an own property with that name doesn't exist, the operation continues to the prototype. If there's no own property on the prototype, then the search continues through the prototype chain until the own property is found or there are no more prototypes to search.

The `has` trap is therefore only called when the search reaches the proxy object in the prototype chain. When using a proxy as a prototype, that only happens when there's no own property of the given name. For example:

```js
let target = {};
let thing = Object.create(new Proxy(target, {
    has(trapTarget, key) {
        return Reflect.has(trapTarget, key);
    }
}));

// triggers the `has` proxy trap
console.log("name" in thing);                   // false

thing.name = "thing";

// does not trigger the `has` proxy trap
console.log("name" in thing);                   // true
```

This code creates a `has` proxy trap on the prototype of `thing`. The `has` trap isn't passed a `receiver` object like the `get` and `set` traps are because searching the prototype happens automatically when the `in` operator is used. Instead, the `has` trap must operate only on `trapTarget`, which is equal to `target`. The first time the `in` operator is used in this example, the `has` trap is called because the property `name` doesn't exist as an own property of `thing`. When `thing.name` is given a value and then the `in` operator is used again, the `has` trap isn't called because the operation stops after finding the own property `name` on `thing`.

The prototype examples to this point have centered around objects created using the  `Object.create()` method. But if you want to create a class that has a proxy as a prototype, the process is a bit more involved.

### Proxies as Prototypes on Classes

Classes cannot be directly modified to use a proxy as a prototype because their `prototype` property is non-writable. You can, however, use a bit of misdirection to create a class that has a proxy as its prototype by using inheritance. To start, you need to create an ECMAScript 5-style type definition using a constructor function. You can then overwrite the prototype to be a proxy. Here's an example:

```js
function NoSuchProperty() {
    // empty
}

NoSuchProperty.prototype = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
});

let thing = new NoSuchProperty();

// throws error due to `get` proxy trap
let result = thing.name;
```

The `NoSuchProperty` function represents the base from which the class will inherit. There are no restrictions on the `prototype` property of functions, so you can overwrite it with a proxy. The `get` trap is used to throw an error when the property doesn't exist. The `thing` object is created as an instance of `NoSuchProperty` and throws an error when the nonexistent `name` property is accessed.

The next step is to create a class that inherits from `NoSuchProperty`. You can simply use the `extends` syntax discussed in Chapter 9 to introduce the proxy into the class' prototype chain, like this:

```js
function NoSuchProperty() {
    // empty
}

NoSuchProperty.prototype = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
});

class Square extends NoSuchProperty {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

let shape = new Square(2, 6);

let area1 = shape.length * shape.width;
console.log(area1);                         // 12

// throws an error because "wdth" doesn't exist
let area2 = shape.length * shape.wdth;
```

The `Square` class inherits from `NoSuchProperty` so the proxy is in the `Square` class' prototype chain. The `shape` object is then created as a new instance of `Square` and has two own properties: `length` and `width`. Reading the values of those properties succeeds because the `get` proxy trap is never called. Only when a property that doesn't exist on `shape` is accessed (`shape.wdth`, an obvious typo) does the `get` proxy trap trigger and throw an error.

That proves the proxy is in the prototype chain of `shape`, but it might not be obvious that the proxy is not the direct prototype of `shape`. In fact, the proxy is a couple of steps up the prototype chain from `shape`. You can see this more clearly by slightly altering the preceding example:

```js
function NoSuchProperty() {
    // empty
}

// store a reference to the proxy that will be the prototype
let proxy = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
});

NoSuchProperty.prototype = proxy;

class Square extends NoSuchProperty {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

let shape = new Square(2, 6);

let shapeProto = Object.getPrototypeOf(shape);

console.log(shapeProto === proxy);                  // false

let secondLevelProto = Object.getPrototypeOf(shapeProto);

console.log(secondLevelProto === proxy);            // true
```

This version of the code stores the proxy in a variable called `proxy` so it's easy to identify later. The prototype of `shape` is `Square.prototype`, which is not a proxy. But the prototype of `Square.prototype` is the proxy that was inherited from `NoSuchProperty`.

The inheritance adds another step in the prototype chain, and that matters because operations that might result in calling the `get` trap on `proxy` need to go through one extra step before getting there. If there's a property on `Square.prototype`, then that will prevent the `get` proxy trap from being called, as in this example:

```js
function NoSuchProperty() {
    // empty
}

NoSuchProperty.prototype = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist`);
    }
});

class Square extends NoSuchProperty {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }
}

let shape = new Square(2, 6);

let area1 = shape.length * shape.width;
console.log(area1);                         // 12

let area2 = shape.getArea();
console.log(area2);                         // 12

// throws an error because "wdth" doesn't exist
let area3 = shape.length * shape.wdth;
```

Here, the `Square` class has a `getArea()` method. The `getArea()` method is automatically added to `Square.prototype` so when `shape.getArea()` is called, the search for the method `getArea()` starts on the `shape` instance and then proceeds to its prototype. Because `getArea()` is found on the prototype, the search stops and the proxy is never called. That is actually the behavior you want in this situation, as you wouldn't want to incorrectly throw an error when `getArea()` was called.

Even though it takes a little bit of extra code to create a class with a proxy in its prototype chain, it can be worth the effort if you need such functionality.

## Summary

Prior to ECMAScript 6, certain objects (such as arrays) displayed nonstandard behavior that developers couldn't replicate. Proxies change that. They let you define your own nonstandard behavior for several low-level JavaScript operations, so you can replicate all behaviors of built-in JavaScript objects through proxy traps. These traps are called behind the scenes when various operations take place, like a use of the `in` operator.

A reflection API was also introduced in ECMAScript 6 to allow developers to implement the default behavior for each proxy trap. Each proxy trap has a corresponding method of the same name on the `Reflect` object, another ECMAScript 6 addition. Using a combination of proxy traps and reflection API methods, it's possible to filter some operations to behave differently only in certain conditions while defaulting to the built-in behavior.

Revocable proxies are a special proxies that can be effectively disabled by using a `revoke()` function. The `revoke()` function terminates all functionality on the proxy, so any attempt to interact with the proxy's properties throws an error after `revoke()` is called. Revocable proxies are important for application security where third-party developers may need access to certain objects for a specified amount of time.

While using proxies directly is the most powerful use case, you can also use a proxy as the prototype for another object. In that case, you are severely limited in the number of proxy traps you can effectively use. Only the `get`, `set`, and `has` proxy traps will ever be called on a proxy when it's used as a prototype, making the set of use cases much smaller.