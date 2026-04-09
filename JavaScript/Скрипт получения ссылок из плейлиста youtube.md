```js
const nodes = document.querySelectorAll('#video-title');
let counter = 1;

for(let i = 0; i < nodes.length; i++){
    if(nodes[i].getAttribute('href') == null) continue
    else {
        console.log(`${counter}) ${nodes[i].getAttribute('title')} - https://youtube.com${nodes[i].getAttribute('href')}`);
        counter++;
    }
}
```