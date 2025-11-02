---const app = require('photoshop').app;

מזהה: "פוטושופ-API"
כותרת: Photoshop API—UXP עבור Adobe Photoshop
תיאור: למד על ה-API של Photoshop שנחשף באמצעות UXP עבור מפתחי תוספים וסקריפטים.
---


# API של Photoshop

## סקירה כללית

השורה הבאה מאפשרת לך גישה לפוטושופ DOM דרך UXP.
```javascript
const app = require('photoshop').app;
```
מכאן תוכלו לפתוח מסמכים, לשנות אותם, להפעיל פריטי תפריט ועוד.

### גרסה מינימלית
כעת תמצא מידע על גרסה מינימלית על מאפיינים ושיטות.  תג גרסה זה מתאים לגרסה של Photoshop שבה החבר הוצג או עודכן לאחרונה באופן משמעותי.
עבור נכסים, תמצא עמודה "MIN VERSION".  עבור שיטות, מספר הגרסה מופיע כתג מימין לשם.


## סינכרוני מול אסינכרוני

הבדל חשוב בין ExtendScript (ו-CEP) ל-UXP בפוטושופ הוא שכל קריאות ExtendScript לפוטושופ היו סינכרוניות. זה אומר שהם חסמו את ממשק המשתמש של Photoshop בזמן שהם בוצעו. ב-UXP, קריאת שיטה היא *אסינכרוני*, ואינו חוסם את שרשור ממשק המשתמש.

למעבר חלק בין ExtendScript DOM ל-UXP DOM, כל המאפיינים (get ו-set) ב-API תוכננו להיות *סינכרוניים* ואין צורך להמתין. ראוי לציין כי הם, ברקע, אסינכרוני באופיים. *synchronous* and do not need to be awaited. It is worth noting that they are, in the background, asynchronous in nature.

## Working with Photoshop Objects

### Photoshop Application

דרך האובייקט [`app`](#overview), תוכל לגשת לשאר האובייקטים והשיטות של Photoshop. [`app`](#overview) object, you can access the rest of Photoshop's objects and methods.

המסמך הפעיל כעת מתקבל כך:

`````` javascript
const doc = app.activeDocument;
```

ואתה יכול לקבל מערך של כל המסמכים הפתוחים כמו זה:

`````` javascript
const allDocuments = app.documents;
```

ראה מאפיינים ושיטות נוספים ב'אפליקציה' תחת [Photoshop](./classes/photoshop/). `app` under [Photoshop](./classes/photoshop/).

### Detour - ExecuteAsModal
מושג מפתח שצריך להבין לפני צלילה ישר לפיתוח תוסף UXP של Photoshop הוא מה שכינינו "ביצוע כמודאלי". כל הפקודות שעשויות **לשנות את המסמך**, או את **מצב היישום**, חייבות להשתמש ב-executeAsModal. **modify the document**, or the **application state**, must utilize executeAsModal.

`````` javascript
async function makeDefaultDocument(executionContext) {
  const app = require('photoshop').app;
  let myDoc = await app.createDocument({preset: "My Web Preset 1"});
}

await require('photoshop').core.executeAsModal(makeDefaultDocument);
```

כפי שאתה עשוי לשים לב, הגבלה זו עשויה להקיף חלק גדול מהפונקציונליות של התוסף שלך! עם זאת, ישנם יתרונות רבים לדגם זה. הסבר מפורט יותר מסופק ב[תיעוד ביצוע כמודאלי](./media/executeasmodal/). [Execute as Modal documentation](./media/executeasmodal/).

### Document
מייצג מסמך פוטושופ בודד ופתוח. מאובייקט זה ניתן לגשת לשכבות, ממדים, רזולוציה וכו' של המסמך. ניתן לחתוך אותו, להוסיף/למחוק/לשכפל שכבות, לשנות גודל, לסובב ולשמור אותו.

קבל את המידות של המסמך הפעיל:

`````` javascript
const app = require('photoshop').app;
const myDoc = app.activeDocument;
const height = myDoc.height;
const width = myDoc.width;
const resolution = myDoc.resolution;
console.log(`Doc size is ${width} x ${height}. Resolution is ${resolution}`);
```

שטח את כל המסמכים הפתוחים כעת:

`````` javascript
const app = require('photoshop').app;
const toFlatten = app.documents;
async function flattenThem(executionContext) {
  toFlatten.forEach((photoshopDoc) => {
    photoshopDoc.flatten();
  });
};

await require('photoshop').core.executeAsModal(flattenThem);
```

Create a layer:
```javascript
const app = require('photoshop').app;
async function newColorDodgeLayer(executionContext) {
  await app.activeDocument.createLayer({ name: "myLayer", opacity: 80, blendMode: "colorDodge" });
};

await require('photoshop').core.executeAsModal(newColorDodgeLayer);
```

See more properties and methods regarding `Document` under [Document](./classes/document/) and [Documents](./classes/documents).

### Layer
Represents a Layer, or a group of Layers. This object is tied to a particular [Document](#Document).

Decrease a layer's opacity and bring it to the front:
```javascript
const app = require('photoshop').app;
const doc = app.activeDocument;
function bringActiveLayerToFront(executionContext) {
  const layer = doc.activeLayers[0];
  layer.opacity = layer.opacity - 10;
  layer.bringToFront();
};

await require('photoshop').core.executeAsModal(bringActiveLayerToFront);
```

Scale down each layer whose name includes 'smaller'
```javascript
const app = require('photoshop').app;
const doc = app.activeDocument;
const layers = doc.layers;
async function scaleLayers(executionContext) {
  for (layer of layers) {
    if (layer.name.includes('smaller')) {
      await layer.scale(80, 80);
    }
  }
};

await require('photoshop').core.executeAsModal(scaleLayers);
```

Note that a layer's `kind` property can be `GROUP` for a Group layer (a layer or folder containing multiple layers). To access the layers in a Group layer, use the `layers` property, and the `parent` property to navigate the layer list tree.

See more properties and methods for `Layer` under [Layer](./classes/layer/) and [Layers](./classes/layers/).

### Actions and ActionSets
Many Photoshop users make heavy use of the `Actions` panel. Actions are essentially macros that can be recorded and played back to script commands and tools that you use frequently. Actions are grouped into `Action Sets`, similar to the way layers can be grouped into Group layers.

The Actions object allows you to delete, duplicate, rename, and play actions. There is no current way to *create* an action using UXP.

Similarly to Actions, the ActionSet object allows you to delete, duplicate, rename, and play Action Sets. There is no current way to *create* an Action Set.

Note that Actions and Action Sets exist app-wide; they're not tied to a specific Document.

Here's an example that finds a particular Action in the default Action Set, then plays it if it exists:

```javascript
const app = require('photoshop').app;
const allActionSets = app.actionTree;
const firstActionSet = allActionSets[0];
let actions = new Map(); // a JS Map allows easy "find by name" operations
firstActionSet.actions.forEach((action) => { actions.set(action.name, action)});
const myAction = actions.get("Wood Frame - 50 pixel");
if (myAction) { // user may have deleted this action
  async function playMyAction(executionContext) {
    await myAction.play();
  }
  await require('photoshop').core.executeAsModal(playMyAction);
}
```

See more properties and methods for `Action` under [Action](./classes/action/) and [ActionSet](./classes/actionset/)

### batchPlay

Photoshop is complex software, with many internal classes and methods. Not all of these are yet exposed via UXP. New interfaces are in development and will be shipped along with each release of Photoshop. In the meantime, if there is something your plugin or script needs to do that is not exposed in the current DOM, you may be able to use `batchPlay`.

BatchPlay is for accessing Photoshop functionality that has not yet been exposed via APIs. BatchPlay is a way to send multiple actions into the Photoshop event queue and return their results.

ExtendScript has `executeAction`; this is analagous to UXP's `batchPlay`. However, whereas `executeAction` could only play one descriptor at a time, `batchPlay` accepts an array of action descriptors. If you have multiple Photoshop operations that need to execute in series, using an array of action descriptors in a single `batchPlay` call is probably what you want.

Unlike ExtendScript where classes were provided to construct action descriptors, references and values, `batchPlay` accepts plain JSON objects.

The [batchPlay documentation](/ps_reference/media/batchplay/) contains details on how to construct JSON for `batchPlay` usage.

## UXP Scripting

UXP is not just for plugins anymore.  Individual JavaScript files may be developed and executed as detailed in the [UXP Scripting section](./media/uxpscripting).
