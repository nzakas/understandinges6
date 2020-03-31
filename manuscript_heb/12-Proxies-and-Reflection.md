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

### מתודות תיאור כפול

שוב, ECMAScript 6 יש כמה מתודות דומות באופן מבלבל, כמו המתודת `Object.defineProperty()` ו `Object.getOwnPropertyDescriptor()` נראה שעושות את הודות הדבר כמו המתודות `Reflect.defineProperty()` ו `Reflect.getOwnPropertyDescriptor()` , בהתאמה. כמו זוגות מתודות אחרות שנדונו קודם לכן בפרק זה, יש להם כמה הבדלים עדינים אך חשובים.

#### המתודה defineProperty()

המתודות `Object.defineProperty()` ו `Reflect.defineProperty()` הם בדיוק אותו הדבר למעט הערך שחוזר מהם. המתודה `Object.defineProperty()` מחזירה את הארומנט הראשון, בזמן ש `Reflect.defineProperty()` תחזיר `true` אם הפעולה הצליחה ו `false` אם לא . לדוגמא:

</div>

```js
let target = {};

let result1 = Object.defineProperty(target, "name", { value: "target "});

console.log(target === result1);        // true

let result2 = Reflect.defineProperty(target, "name", { value: "reflect" });

console.log(result2);                   // true
```
<div dir="rtl">

כאשר `Object.defineProperty()` נקראת על `target`, הערך החוזר הוא `target`. כאשר `Reflect.defineProperty()` נקרא על `target`, הערך החוזר הוא `true`, שמעיד שהפעולה הצליחה. מאחר ומלכודת הפרוקסי `defineProperty` דורשת שערך בוליאני יחזור, זה יותר טוב להשתמש ב `Reflect.defineProperty()` ליישם את התנהגות ברירת המחדל במידת הצורך.

#### המתודה getOwnPropertyDescriptor()

המתודה `Object.getOwnPropertyDescriptor()` כופה את vtrdunby הראשון שלו לאובייקט כאשר מועבר ערך פרימיטיבי ואז ממשיך בפעולה. מצד שני, המתודה `Reflect.getOwnPropertyDescriptor()` תזרוק שגיאה אם הארגומנט הראשון יהיה מסוג פרמטיבי. להלן דוגמה המציגה את שניהם:

</div>

```js
let descriptor1 = Object.getOwnPropertyDescriptor(2, "name");
console.log(descriptor1);       // undefined

// יזרוק שגיאה
let descriptor2 = Reflect.getOwnPropertyDescriptor(2, "name");
```

<div dir="rtl">

המתודה `Object.getOwnPropertyDescriptor()` תחזיר `undefined` בגלל ההשמה של `2` לתוך אובייקט, ולו אין את המאפיין `name` . זוהי ההתנהגות הסטנדרטית של המתודה כאשר מאפיין עם השם הנתון אינו נמצא באובייקט. כאשר `Reflect.getOwnPropertyDescriptor()` נקרא, מצד שני, שגיאה נזרקת מייד מכיוון שמתודה זו אינה מקבלת ערכים פרימיטיביים לטיעון הראשון.

## מלכודת `ownKeys`

מלכודת הפרוקסי `ownKeys` מיירטת את המתודה הפנימית  `[[OwnPropertyKeys]]` ומאפשרת לך לשנות את התתנהגות שלה ע"י החזרה של מערך של ערכים. במערך הזה נעשה שימוש בארבע מתודות שונות: המתודה `Object.keys()` , המתודה `Object.getOwnPropertyNames()` , המתודה `Object.getOwnPropertySymbols()`, והמתודה `Object.assign()` . (המתודה `Object.assign()` משתמשת במערך כדי להחליט איזה מאפיינים להעתיק.)

ההתנהגות הדיפולטיבית עבור המלכודת `ownKeys` ממומשת ע"י המתודה `Reflect.ownKeys()` ומחזירה מערך של כל מפתחות של המאפיינים,כולל סטרינג וסימבול. המתודה `Object.getOwnProperyNames()` והמתודה `Object.keys()` מסננות סימבול מהמערך ומחזירה את התוצאות בזמן שהמתודה  `Object.getOwnPropertySymbols()` מסננת את הסטרינג ומחזירה את התוצאות. המתודה `Object.assign()` משתמשת גם בסטרינג וגם בסימבול.

המלכודת `ownKeys` מקבלת ארגומנט אחד בלבד, המטרה, וחייבת להחזיר תמיד מערך או מערך-דמה של אובייקט (array-like); אחרת, שגיאה תיזרק. אתה יכול להשתמש במלכודת  `ownKeys` לדוגמא, לסנן החוצה את כל המפתחות שאתה לא רוצה שישתמשו בזמן הקריאה  למתודה `Object.keys()`, למתודה `Object.getOwnPropertyNames()`, למתודה `Object.getOwnPropertySymbols()`, או למתודה `Object.assign()`. נניח שאינך רוצה לכלול את המאפיינים שמתחילים עם קו-תחתון, סימון נפוץ ב- JavaScript המצביע על כך ששדה הוא פרטי. אתה יכול להשתמש במלכודת `ownKeys` לסנן החוצה את המפתחות האלו כך:

</div>

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

<div dir="rtl">

הדוגמה הזו השתמשנו במלכודת `ownKeys` שקוראת בהתחלה למתודה `Reflect.ownKeys()` כדי לקבל את כל המפתחות של אובייקט המטרה. ואז במתודה `filter()` נעשה שימוש לסנן החוצה את המפתחות שמתחילות עם סימון של קו-תחתון. לאחר מכן מוספים שלושה מאפיינים לאובייקט פרוקסי `proxy` : `name`, `_name`, ו `nameSymbol`. כאשר `Object.getOwnPropertyNames()` ו `Object.keys()` נקראות על פרוקסי `proxy`, רק המאפיין `name` מוחזר. באופן דומה `nameSymbol` מוחזר כאשר `Object.getOwnPropertySymbols()` נקרא על  `proxy`. המאפיין `_name` לא מופיע בשום תומאה מאחר והוא סונן החוצה.

I> המלכודת `ownKeys` משפיעה גם על הלולאה `for-in`, הקוראת למלכודת כדי לקבוע באילו מפתחות להשתמש בתוך הלולאה.

## פונקמיות פרוקסי עם מלכודות `apply` ו `construct` 

מבין כל מלכודות הפרוקסי, רק `apply` ו `construct` דורשים השמטרה של הפרוקסי התיה פונקציה. נזכיר מפרק 3 שלפונקציות שתי מתודות פנימיות הנקראות `[[Call]]` ו `[[Construct]]` המבוצעות כאשר פונקציה נקראת ללא ועם האופרטור `new` , בהתאמה. המלכודות `apply` ו `construct` תואמים ומאפשרים לך לעקוף את אותן המתודות הפנימיות. כאשר פונקציה נקראית בלי באופרטוק `new`, המלכודת `apply` ו `Reflect.apply()` מקבלים ומצפים , לארגומנטים הבאים :


1. `trapTarget` - הפונקציה שמבוצעת (the proxy's target)
1. `thisArg` - הערך של  `this` בתוך הפונקציה במהלך הקריאה
1. `argumentsList` - מערך של ארגומנטים שהועברו לפונקציה 

מלכודת `construct` , אשר נקרא כאשר הפונקציה מבוצעת באמצעות `new`, מקבלת את הארגומנטים הבאים:

1. `trapTarget` - הפונקציה שמבוצעת (the proxy's target)
1. `argumentsList` - מערך של ארגומנטים שהועברו לפונקציה

המתודה `Reflect.construct()` מקבלת גם את שני הארגומנטים הללו ויש להם ארגומנט שלישי אופציונלי שנקרא `newTarget`. כאשר ניתן, הארגומנט `newTarget` מציין את הערך של `new.target` בתוך הפונקציה.

יחד, מלכודות `apply` ו `construct` שולטות באופן מוחלט בהתנהגות של כל פונקציית יעד של פרוקסי. בשביל לחקות את התנהגות ברירת המחדל של פונקציה, אתה יכול לעשות זאת:

</div>

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

// פרוקסי עם פונקציה כמטרה שלו נראה כמו פונקציה
console.log(typeof proxy);                  // "function"

console.log(proxy());                       // 42

var instance = new proxy();
console.log(instance instanceof proxy);     // true
console.log(instance instanceof target);    // true
```

<div dir="rtl">

בדוגמה זו יש פונקציה שמחזירה את המספר 42. ה- proxy עבור פונקציה זו משתמש במלכודות `apply` ו `construct` להאציל התנהגויות אלה למתודות `Reflect.apply()` ו `Reflect.construct()`, בהתאמה. התוצאה הסופית היא שפונקציית ה- Proxy פועלת בדיוק כמו פונקציית היעד, כולל זיהוי עצמו כפונקציה כאשר משתמשים ב- `typeof`. כאשר הפרוקסי נקרא בלי `new` הוא יחזיר 42 וכאשר נקרא עם  `new` הוא יצור אובייקט שנקרא `instance`. האובייקט `instance` נחשב למופע של שניהם- של `proxy` ו `target` בגלל `instanceof` משתמש בשרשרת אב-הטיפוס (prototype chain) כדי לקבוע מידע זה. בדיקת שרשרת אב-טיפוס אינה מושפעת מהפרוקסי, וזו הסיבה ש-  `proxy` ו `target` נראים בעלי אב-טיפוס זהה למנוע ה- JavaScript.

### אימות פרמטרים של פונקציה

המלכודות `apply` ו `construct` לפתוח הרבה אפשרויות לשינוי אופן ביצוע הפונקציה. לדוגמה, נניח שברצונך לאמת שכל הארגומנטים הם מסוג מסוים. אתה יכול לבדוק את הטיעונים במלכודת `apply`:

</div>

```js
// מוסיף יחד את כל הארגומנטים
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

// יזרוק שגיאה
console.log(sumProxy(1, "2", 3, 4));

// גם יזרוק שגיאה
let result = new sumProxy();
```

<div dir="rtl">

בדוגמא הזו שנו משתמשים במלכודת `apply` להבטיח שכל הארגומנטים הם מספרים. הפונקציה `sum()`  מחברת את כל הפרמטרים המועברים. אם מועבר ערך שאינו מספר, הפונקציה עדיין תנסה לבצע את הפעולה, מה שעלול לגרום לתוצאות בלתי צפויות. ע"י זה שאנחנו עוטפים את `sum()` בתוך הפרוקסי `sumProxy()` , קוד זה מיירט קריאות לפונקציה ומבטיח שכל ארגומנט הוא מספר לפני שהוא מאפשר לקריאה להמשיך. כדי להיות בטוחים, הקוד משתמש גם במלכודת `construct` כדי להבטיח שלא ניתן לקרוא לפונקציה עם `new`.

אתה יכול גם לעשות את ההפך, להבטיח שחייבים לקרוא לפונקציה עם `new` ולאמת את הארגומנטים שלה שיהיו מספרים :

</div>

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

// יזרוק שגיאה
NumbersProxy(1, 2, 3, 4);
```

<div dir="rtl">

כאן, מלכודת `apply` יזרוק שגיאה בזמן שמלכודת `construct` משתמשת במתודה `Reflect.construct()` כדי לאמת קלט ולהחזיר מופע חדש. כמובן, שתוכלו להשיג את אותו הדבר מבלי להשתמש בפרוקסי ע"י שימוש ב `new.target` במקום.

### לקרוא לקונסטרקטור-Constructors בלי new

בפרק 3 הצגנו את המטא-מאפיין `new.target` metaproperty. נזכיר, `new.target` היא הפניה לפונקציה עליה `new` נקרא, כלומר תוכלו לדעת אם נקראה פונקציה באמצעות `new` או לא על ידי בדיקת הערך של `new.target` כך:

</div>

```js
function Numbers(...values) {

    if (typeof new.target === "undefined") {
        throw new TypeError("This function must be called with new.");
    }

    this.values = values;
}

let instance = new Numbers(1, 2, 3, 4);
console.log(instance.values);               // [1,2,3,4]

// יזרוק שגיאה
Numbers(1, 2, 3, 4);
```
<div dir="rtl">

דוגמה זו זורקת שגיאה מתי ש `Numbers` נקראית בלי `new`, הדומה לדוגמא בסעיף "אימות פרמטרים של פונקציה" אבל בלי שימוש עם פרוקסי. כתיבת קוד כזה היא הרבה יותר פשוטה מאשר שימוש בפרוקסי ועדיפה אם המטרה היחידה שלך היא למנוע קריאה לפונקציה בלי `new`. אך לפעמים אינך שולט בפונקציה אשר יש לשנות את התנהגותה. במקרה זה, השימוש בפרוקסי הגיוני.

נניח שהפונקציה `Numbers` מוגדר בקוד שלא ניתן לשנות. אתה יודע שהקוד מסתמך על `new.target` וברצונך להימנע מאותה בדיקה בזמן שאתה קורא לפונקציה. ההתנהגות בעת השימוש `new` נקבעה כבר, כך שתוכלו פשוט להשתמש במלכודת `apply` :

</div>

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

<div dir="rtl">

הפונקציה `NumbersProxy` מאפשרת לך לקרוא ל `Numbers` מבלי להשתמש ב `new` ושהיא תתנהג כאילו השתמשה ב `new`. לשם כך, המלכודת `apply` קוראית ל `Reflect.construct()` עם הארגומנטים המועוברים ל `apply`. ה `new.target` בתוך `Numbers` הוא שווה ל `Numbers` עצמו, ןהשגיאה תיזרק. אמנם זו דוגמה פשוטה לשינוי `new.target`, אתה יכול גם לעשות זאת באופן ישיר יותר.

### עקיפת מחלקה אבסטרקטית

אתה יכול ללכת צעד אחד קדימה ולפרט את הארגומנט השלישי ב `Reflect.construct()` כערך ספציפי שיש להקצות עבור `new.target`. זה יכול להיות יעיל כאשר הפונקציה בודקת את  `new.target` מול ערך ידוע, כמו בעת יצירת קוסנרקטור לקלאס בסיס אבסטרקטי (נדון בפרק 9). בקונסטרקטור בקלאס אבסטקטי (מופשט) In , `new.target` מצפה להיות משהו אחר מאשר הקוסנטרקטור עצמו , כמו בדוגמא:

</div>

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

// יזרוק שגיאה
new AbstractNumbers(1, 2, 3, 4);
```

<div dir="rtl">

כאשר `new AbstractNumbers()` נקרא, `new.target` שווה ל `AbstractNumbers` ושגיאה נזרקת. שקוראים ל `new Numbers()` יעבוד כי  `new.target` שווה ל `Numbers`.אתה יכול לעקוף מגבלה זו על ידי הקצאה ידנית `new.target` עם פרוקסי:

</div>

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

<div dir="rtl">

ה `AbstractNumbersProxy` משתמש במלכודת `construct` ליירט את הקריאה למתודה `new AbstractNumbersProxy()` . כאשר המתודה  `Reflect.construct()` נראית עם ארגומנט מהמלכות ומוסיפים כארגומנט שלישי פונקציה ריקה. הפונקציה הריקה משמשת כערך  ל `new.target` במקום הבנאי- הקונסרקטור. בגלל ש `new.target` לא שווה ל  `AbstractNumbers`, לא נזרקית שגיאה והבנאי מופעל בשלמות.

### החלפת קונסטרקטור-בנאי של קלאס

פרק 9 הסביר שבנאי שקלאס קונסטקטור חייב להיקרא עם `new`. זה קורה בגלל המתודה הפנימית `[[Call]]` שהיא עבור קלאס קונסטרקטור שמוגדרת לזרוק שגיאה. אבל פרוקסי יכול ליירט קריאות למתודה `[[Call]]`,זאת אומרת שאת היכול ליצור ביעילות בנאי של קונסטרקטור על ידי שימוש בפרוקסי.לדוגמא,אם אתה רוצה שקלאס קונסטרקטור יעבוד בלי `new`, אתה יכול להשתמש במלכודת `apply` על מנת ליצור ישות חדשה . הנה קוד לדוגמא:

</div>

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

<div dir="rtl">

האובייקט `PersonProxy` הוא פרוקסי של קלאס-קונסטרקטור `Person`.  קלאס קונסטרקטור הוא סכ"ה פונקציה,אז ההתנהגות שלו תהיה כמו פונקציה שנשתמש בפרוקסי.מלכודת  `apply` היא תשכתב את ההתנהגות הדיפולטיבית ובבמקום זאת תחזיר אינסטנס חדש של `trapTarget` שזה שווה ערך ל  `Person`. (בדוגמא זאת אני משתמש ב `trapTarget`  להראות שאתה לא צריך ידנית להגדיר קלאס.) ה `argumentList` מועבר ל `trapTarget` בשימוש ה spread operator להעברת הארגונמטים. קריאה ל  `PersonProxy()` בלי להשתמש `new` יחזיר אינטנס של  `Person`; אם אתה תנסה לקרוא ל  `Person()` בלי `new`, הקונסטרקטור עדיין יזרוק שגיאה. ליצור אפשרות לקרוא לקלאס קונסטקרטוד זה משהו שאפשרי רק ע"י שימוש בפרוקסי.

## לבטל פרוקסי

בדרך כלל פרוקסי לא יכול להתנתק מהמטרה שלו ארי הוא נוצר. כל הדוגמאות בפרק זה השתמשנו בפרוקסים בלתי ניתנים לביטול. אך יתכנו מצבים שבהם ברצונך לבטל פרוקסי כך שלא ניתן יהיה להשתמש בו עוד. תמצא שזה מועיל ביותר לבטל פרוקסי עבודה כאשר ברצונך לספק אובייקט באמצעות API למטרות אבטחה ולשמור על היכולת לנתק את הגישה לפונקציונליות כלשהי בכל נקודת זמן.

אתה יכול ליצור פרוקסי הניתן לביטול ע"י המתודה `Proxy.revocable()` , שלוקחת את אותם ארגומנטים כמו הבנאי של  ה `Proxy` -- אובייקט מטרה ואת המטפל של הפרוקסי (handler). ערך ההחזרה הוא אובייקט עם המאפיינים הבאים:

1. `proxy` - אובייקט ה- proxy שניתן לבטל
1. `revoke` - הפונקציה להתקשר לביטול ה- Proxy

כאשר הפונקציה `revoke()` נקראית, לא ניתן לבצע פעולות נוספות דרך `proxy`. כל ניסיון לקיים אינטראקציה עם אובייקט ה- proxy יגרום למלכודת proxy לזרוק שגיאה. לדוגמא:

</div>

```js
let target = {
    name: "target"
};

let { proxy, revoke } = Proxy.revocable(target, {});

console.log(proxy.name);        // "target"

revoke();

// יזרוק שגיאה
console.log(proxy.name);
```

<div dir="rtl">

בדוגמה הזו יצרנו פרוקסי הניתן לביטול. השתמשנו ב destructuring להשמה של המשתנים `proxy` ו `revoke` לתכונות עם אותו שם שהחוזרו ע"י המתודה `Proxy.revocable()`. לאחר מכן,ניתן להשתמש  באובייקט `proxy` כמו כל פרוקסי שלא ניתן לביטול, כך ש `proxy.name` יחזיר `"target"` כי זה עובר ל `target.name`. כאשר הפונקציה `revoke()` נקראית, מאידך, `proxy` כבר לא מתפקד. נסיון לגשת ל  `proxy.name` יזרוק שגיאה, כמו כל פעולה אחרת שתפעיל מלכודת `proxy`.

## לפתור את בעיית המערך

בתחילת פרק זה הסברתי כיצד מפתחים לא יכולים לחקות את ההתנהגות של מערך במדויק ב- JavaScript לפני ECMAScript 6. פרוקסי והAPI של ההשתקפות (reflection API) מאפשרים לך ליצור אובייקט שמתנהג כמו הסוג המובנה של `Array` כאשר מוסיפים או מחסירים ערכים. לריענון הזיכרון, הנה מספר דוגמאותשפרוקסי יכול לחקות:

</div>

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

<div dir="rtl">

בדוגמה זו ישנן שתי התנהגויות חשובות במיוחד:

1. המאפיין  `length` יוגדל לארבע כאשר תהיה השמה של ערך ב  `colors[3]`.
1. שני הערכים האחרונים של המערך ימחקו כאשר המאפיין `length` מוגדר ל 2.

שתי ההתנהגויות הללו הן היחידות שצריך לחקות כדי לשחזר במדויק את האופן שבו מערכים מובנים עובדים. החלקים הבאים הבאים מתארים כיצד ליצור אובייקט שמחקה אותם נכון.

### איתור מדדי מערך

קחו בחשבון שהקצאה למפתח מספר שלם הוא מקרה מיוחד עבור מערכים, מכיוון שאלו מטופלים באופן שונה ממפתחות שאינם מספרים. מפרט ECMAScript 6 נותן הוראות אלה כיצד לקבוע אם ערך מפתח הוא אינדקס מערך:

> A String property name `P` is an array index if and only if `ToString(ToUint32(P))` is equal to `P` and `ToUint32(P)` is not equal to 2^32^-1.

ניתן ליישם פעולה זו ב- JavaScript באופן הבא:

</div>

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}
```

<div dir="rtl">

הפונקציה `toUint32()` ממירה ערך שמתקבל ממיר ערך נתון למספר שלם של  32 סיביות לא חתום באמצעות אלגוריתם המתואר במפרט. הפונקציה `isArrayIndex()` ראשית ממיר את המפתח ל- uint32 ואז מבצע את ההשוואות כדי לקבוע אם המפתח הוא אינדקס מערך או לא. עם פונקציות שירות אלה זמינות, אתה יכול להתחיל ליישם אובייקט שיחקה מערך מובנה.

### הגדלת האורך בעת הוספת אלמנטים חדשים

אולי שמתם לב ששתי התנהגויות המערך שתיארתי מסתמכות על הקצאת ערך. זה אומר שאתה באמת צריך להשתמש רק במלכודת `set` של פרוקסי להשיג את ההתנהגות הזו. כדי להתחיל, הנה דוגמה המיישמת את ההתנהגות הראשונה משתיים על ידי הגדלת המאפיין `length` כאשר משתמשים באינדקס מערך גדול מ `length - 1` :

</div>

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

            // המקרה המיוחד
            if (isArrayIndex(key)) {
                let numericKey = Number(key);

                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, "length", numericKey + 1);
                }
            }

            // עשה זאת תמיד ללא קשר לסוג המפתח
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

<div dir="rtl">

הדוגמא הזו משתמשת במלכודת פרוקסי `set` ליירט הגדרת אינדקס מערך. אם המפתח הוא אינדקס מערך, הוא יומר למספר מכיוון שמפתחות מועברים תמיד כמחרוזות. בשלב הבא, אם הערך המספרי הזה גדול או שווה למאפיין `length` הנוכחי, אז המאפיין `length` מתעדכן להיות אחד יותר מהמפתח המספרי (הגדרה של פריט במיקום 3 פירושה ש `length` חייב להיות 4). לאחר מכן משתמשים בהתנהגות ברירת המחדל להגדרת מאפיין באמצעות  `Reflect.set()`, מכיוון שאתה רוצה שהמאפיין יקבל את הערך כמפורט.

המערך המותאם אישית מאותחל על ידי קריאה ל `createMyArray()` עם  `length` של 3 והערכים לשלושת הפריטים מתווספים מיד לאחר מכן. המאפיין `length` נכון לעכשיו נשאר 3 עד שהערך `"black"` מוקצה למיקום  3. בנקודה הזו, `length` מוגדר ל 4.

כאשר ההתנהגות הראשונה עובדת, הגיע הזמן לעבור לשנייה.

### מחיקת אלמנטים כאשר מצמצמים את האורך - length

ההתנהגות הראשונה לחיקוי המערך תהיה רק כאשר  האינדקס של המערך יהיה שווה או גדול מהערך של המאפיין `length`. ההתנהגות השנייה עושה את ההפך ומסירה פריטי מערך כאשר מאפיין `length` מוגדר לערך קטן יותר מכפי שהיה קודם. זה כרוך לא רק בשינוי מאפיין `length` , אלא גם במחיקת כל הפריטים שעלולים להתקיים. לדוגמא אם מערך עם `length` של 4 וה `length` מוגדר ל 2, האייטמים במקומות  2 ו 3 ימחקו. אתה יכול להשיג זאת בתוך מלכודת הפרוקסי `set` לצד ההתנהגות הראשונה. הנה הדוגמה הקודמת שוב, עם עידכון למתודה `createMyArray`:

</div>

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

            // המקרה המיוחד
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

            // עשה זאת תמיד ללא קשר לסוג המפתח
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

<div dir="rtl">

המלכודת `set` בקוד הזה בודקת אם `key` הוא `"length"` על מנת להתאים נכון את שאר האובייקט.כאשר זה קורה, האורך הנוכחי נשלף תחילה באמצעות `Reflect.get()` ומושווה לערך החדש. אם הערך החדש הוא קטן מהאורך הנוכחי, אזי הלופ `for`מוחק את כל המאפיינים ביעד שאינם אמורים להיות זמינים עוד. הלופ `for` הולך לאחור מאורך המערך הנוכחי (`currentLength`) ומוחק כל נכס עד שהוא מגיע לאורך המערך החדש (`value`).

דוגמה זו מוסיפה ארבעה צבעים ל `colors` ואז קובעת את המאפיין `length` ל 2. זה מסיר ביעילות את הפריטים בעמדות 2 ו -3, כך שהם חוזרים כעת `undefined` שמנסים לגשת אליהם. המאפיין `length` המאפיין מוגדר כ- 2 והפריטים בעמדות 0 ו- 1 עדיין נגישים.

כאשר שתי ההתנהגויות מיושמות, תוכלו ליצור בקלות אובייקט המחקה את ההתנהגות של מערכים מובנים. אולם פעולה כזו עם פונקציה אינה רצויה כמו יצירת מחלקה שתכסה את ההתנהגות הזו, אז השלב הבא הוא ליישם פונקציונליות זו כמחלקה-קלאס.

### הטמעת מחלקת MyArray

הדרך הפשוטה ביותר ליצור מחלקה המשתמשת בפרוקסי היא להגדיר את המחלקה כרגיל ואז להחזיר פרוקסי מהבנאי. באופן זה, האובייקט המוחזר כאשר מופעלת המחלקה יהיה ה- proxy במקום המופע. (המופע הוא הערך של `this` בתוך הבנאי.) המופע הופך להיות היעד של ה- proxy וה- proxy מוחזר כאילו היה המופע. המופע יהיה פרטי לחלוטין ולא תוכלו לגשת אליו ישירות, אם כי תוכלו לגשת אליו בעקיפין דרך ה- proxy.

להלן דוגמא פשוטה להחזרת שרת פרוקסי מבונה כיתתי:

</div>

```js
class Thing {
    constructor() {
        return new Proxy(this, {});
    }
}

let myThing = new Thing();
console.log(myThing instanceof Thing);      // true
```

<div dir="rtl">

בדוגמא הזו המחלקה `Thing` מחזירה פרוקסי מתוך הבנאי. המטרה של הפרוקסי היא `this` והפרוקסי מוחזר מהבנאי. הכוונה ש `myThing` הוא למעשה פרוקסי למרות שזה נוצר על ידי הקריאה לבנאי של `Thing`. בגלל שפרוקסים מעבירים דרך ההתנהגות שלהם אל היעד שלהם, `myThing` נחשב עדיין למופע של `Thing`, מה שהופך את ה- Proxy לשקוף לחלוטין לכל מי שמשתמש במחלקה `Thing` .

עם זאת בחשבון, יצירת כיתת מערך בהתאמה אישית באמצעות פרוקסי הוא יחסית פשוט. הקוד זהה בעיקר לקוד בסעיף "מחיקת אלמנטים על צמצום האורך". משתמשים באותו קוד פרוקסי, אך הפעם הוא נמצא בתוך בנאי כיתתי. להלן הדוגמא המלאה:

</div>

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

                // המקרה המיוחד
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

                // עשה זאת תמיד ללא קשר לסוג המפתח
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

<div dir="rtl">

הקוד הזה יצר מחלקה `MyArray` שמחזירה פרוקסי מתוך הבנאי שלה.המאפיין `length` מתווסף בתוך הבנאי (מאתחל לערך שמועבר או לערך ברירת מחדל של 0) והפרוקסי נוצר ומוחזר. זה נותן למשתנה `colors` את הנראות להיות רמופע של `MyArray` ומיישם את שתי התנהגויות של המערך והמפתח.

אמנם קל להחזיר פרוקסי מבנאימחלקה, אך זה אומר שנוצר פרוקסי חדש עבור כל מופע. עם זאת יש דרך לגרום לכל המקרים לשתף פרוקסי אחד: אתה יכול להשתמש בפרוקסי כאב-טיפוס - prototype.

## שימוש בפרוקסי כאב-טיפוס - Prototype

פרוקסיות יכולות לשמש כאבות-טיפוס, אך פעולה זו מסובכת מעט יותר מהדוגמאות הקודמות בפרק זה. כאשר פרוקסי הוא אב-טיפוס, מלכודות ה- Proxy נקראות רק כאשר פעולת ברירת המחדל תמשיך בדרך כלל לאב-טיפוס, מה שמגביל את יכולות ה- Proxy כאב-טיפוס. שקול דוגמה זו:

</div>

```js
let target = {};
let newTarget = Object.create(new Proxy(target, {

    // לעולם לא יקרא
    defineProperty(trapTarget, name, descriptor) {

        // יחזיר שגיאה אם יקרא
        return false;
    }
}));

Object.defineProperty(newTarget, "name", {
    value: "newTarget"
});

console.log(newTarget.name);                    // "newTarget"
console.log(newTarget.hasOwnProperty("name"));  // true
```

האובייקט `newTarget` נוצר באמצעות פרוקסי כאב-טיפוס. הפיכת `target` למטרת הפרוקסי באופן יעיל עושה  את `target` לאב-טיפוס של `newTarget` בגלל שהפרוקסי הוא שקוף. כעת, מלכודות פרוקסי יקראו רק אם פעולה ב- `newTarget` תעביר את הפעולה לקרות ב `target`.

המתודה `Object.defineProperty()` נקראית ב `newTarget` ליצירת מאפיין משלו בשם `name`. הגדרת מאפיין על אובייקט אינה פעולה שממשיכה בדרך כלל לאב-טיפוס של האובייקט, כך שמלכודת  `defineProperty` על הפרוקסי לעולם לא נקראת והמאפיין `name` מתווסף ל `newTarget` כמאפיין משל עצמו.

בעוד שיכולות פרוקסי  מוגבלים מאוד כאשר משתמשים בהם כאבות-טיפוס, ישנם כמה מלכודות שעדיין מועילות.

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