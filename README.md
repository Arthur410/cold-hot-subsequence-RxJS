# Cold and Hot subsequence on RxJS
Task: The user enters the site and internal listening begins (any request to the server to receive data, in our case it will be the usual sleepAsync - a function with a timeout). There are two listening modes - optimized (we'll call it hot) and constant (we'll call it cold). If the user is active on the tab, we enable the optimized method; if the user left the tab, we enable the constant method. You also need to show the amount of time that the wiretap took.

Задача: Пользователь заходит на сайт и начинается внутренняя прослушка (какой либо запрос на сервер с получением данных, в нашем же случае будет обычный sleepAsync - функция с таймаутом). Режимов прослушки два - оптимизированный (будем называть горячим) и константный (будем называть холодным). При условии активности пользователя на вкладке, включаем оптимизированный метод, если же пользователь покинул вкладку, включаем константный. Также нужно показывать количество времени, которое занимала прослушка.

![изображение](https://github.com/Arthur410/cold-hot-subsequence-RxJS/assets/67192703/0eec1507-a2eb-43cd-81e0-a30c2aaa47fe)


## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
