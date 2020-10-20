# vue-minesweeper

|**Description**|
|:---|
|Minesweeper game made in Vue|
|**Dependencies**|
|`"core-js": "^3.6.5"`</br>`"fontawesome": "^5.6.3"`</br>`"node-sass": "^4.14.1"`</br>`"sass-loader": "^10.0.2"`</br>`"vue": "^2.6.11"`|
|**Setup** (via `npm`)|
|`~$ npm install` to install dependencies<br/>`~$ npm run serve` to run a local instance|
|**Demo**|
|![](demo.gif)|

|data|desc|
|:---|:---|
|`width`|Current width|
|`height`|Current height|
|`state`|(0) Game in progress </br> (1) Won </br> (-1) Lost|
|`minefield`|Minefield array of objects|
|`mineCount`|Count of mines deployed|
|`mineProbability`|Current probability|
|`flagsPlanted`|Count of flags planted by player|
|`minefieldLock`|Click lock|
|`difficulties`|Array of objects with difficulty data|

|methods|desc|
|:---|:---|
|setDifficulty(difficulty)|Sets the difficulty</br>(difficulty: object defined in `difficulties`)|
|newMinefield(w, h)|Creates a new minefield</br>(w: width, h: height)|
|fieldLeftClick(index)|Click event handler for LMB</br>(index: field index)|
|fieldRightClick(index)|Click event handler for RMB</br>(index: field index)|
|gameOver(index)|You die|
|gameSuccess|You don't die|
|goBack|You surrender|