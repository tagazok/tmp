# Steps


<div style="color: red; font-style: italic;">
<pre>
gg1
sw.addSyncEventListener
</pre>
</div>

```javascript
self.addEventListener('sync', event => {
  if (event.tag === 'postPhoto') {
    console.log('sync event fired')
  }
});
```

<div style="color: red; font-style: italic">
<pre>
gg2
cl.registerSync
</pre>
</div>

```javascript
if (reg) {
  reg.sync.register('postPhoto').then(() => {
    console.log('Sync registered');
  });
}
```

Show slides: What are we going to try to achieve ?
idb for IndexedDB

<div style="color: red; font-style: italic">
<pre>
gg3
html.htmlsnipper
</pre>
</div>

```html
<script src='assets/js/vendors/idb.js'></script>
<script src='assets/js/datastore.js'></script>
```

Create Datastore class
```javascript
class  {
  constructor() {
    
  }
}
```

<div style="color: red; font-style: italic">
<pre>
gg4
dts.idb.open
</pre>
</div>

```javascript
this.dbPromise = idb.open('offlinedemo-db', 1, upgradeDb => {
  upgradeDb.createObjectStore('photobooth', {
    keyPath: 'id'
  });
});
```

<div style="color: red; font-style: italic">
<pre>
gg5
dts.getstore.fn
</pre>
</div>

```javascript
async getStore() {
  const db = await this.dbPromise;
  return db.transaction('photobooth', 'readwrite').objectStore('photobooth');
}
```

In client, create a Datastore object
```javascript
const ds = new Datastore();
```

Test : refresh. We see the database created

In addPhoto. Instead of sending to internet, we add to the database
<div style="color: red; font-style: italic">
<pre>
gg6
cl.getstore.call
</pre>
</div>

```javascript
ds.getStore().then(store => {
  store.put(photo);
  photos_list.insertBefore(generatePhotoTemplate(photo, false), photos_list.firstChild);
});
```
Test : Photos are added to the database.
Test : Refresh, we don't see the photos

Let's create a method to retrieve all the photos from the database
<div style="color: red; font-style: italic">
<pre>
gg7
dts.getallphotosfromdb.fn
</pre>
</div>

```javascript
async getAllPhotosFromDb() {
  const store = await this.getStore();
  return store.getAll();
}
```

And let's call it from our client.
<div style="color: red; font-style: italic">
<pre>
gg8
cl.getallphotosfromdb.call
</pre>
</div>

```javascript
ds.getAllPhotosFromDb().then(data => {
  for (let photo of data) {
    photos_list.insertBefore(generatePhotoTemplate(photo, false), photos_list.firstChild);
  }
});
```

Test. Take photos, refresh, here they are :)


## Let part done! Let's do right part

<div style="color: red; font-style: italic">
<pre>
gg11
sw.importscripts
</pre>
</div>

```javascript
self.importScripts('./assets/js/vendors/idb.js');
self.importScripts('./assets/js/datastore.js');
```

<div style="color: red; font-style: italic">
<pre>
gg9
sw.uploadphotos.call
</pre>
</div>

```javascript
event.waitUntil(uploadPhotos());
```

<div style="color: red; font-style: italic">
<pre>
gg10
sw.uploadphotos.fn
</pre>
</div>

```javascript
async function uploadPhotos() {
  const photos = await ds.getAllPhotosFromDb();
  for (let photo of photos) {
    try {
      await uploadPhoto(photo);
    await removePhoto(photo);
    } catch (error) {
      console.log(`Error : Can't upload photo error`);
    }
  }
}
```

Ne pas mettre les async/await tout de suite
<div style="color: red; font-style: italic">
<pre>
gg12
sw.uploadphoto.fn
</pre>
</div>

```javascript
function uploadPhoto(photo) {
  fetch('https://offline-demo-4b2ee.firebaseio.com/photos.json', {
    method: 'post',
    body: JSON.stringify(photo)
  });
}
```

Test. Take photos, they uplad, take another one. all re-upladed
Issue: We need to remove photos once upladed


<div style="color: red; font-style: italic">
<pre>
gg13
sw.removephoto.fn
</pre>
</div>

```javascript
async function removePhoto(photo) {
  const store = await ds.getStore();
  store.delete(photo.id);
}
```

Would be nice to change icons

<div style="color: red; font-style: italic">
<pre>
gg15 - gg16
sw.broadcast.call.backup - sw.broadcast.call.done
</pre>
</div>

```javascript
broadcast('backup', photo.id);
broadcast('cloud_done', photo.id);
```


<div style="color: red; font-style: italic">
<pre>
gg14
sw.broadcast.fn
</pre>
</div>

```javascript
async function broadcast(action, photoId) {
  const clients = await self.clients.matchAll();
  for (let client of clients) {
    client.postMessage({
      client: client.id,
      message: {action: action, id: photoId}
    });
  }
}
```

<div style="color: red; font-style: italic">
<pre>
gg17
cl.broadcast.register
</pre>
</div>

```javascript
navigator.serviceWorker.addEventListener('message', event => {
  let message = event.data.message;
  document.querySelector(`#photo-${message.id} .material-icons`).textContent = message.action;
});
```
