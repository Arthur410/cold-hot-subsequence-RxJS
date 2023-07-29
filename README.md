# Cold and Hot subsequence on RxJS
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
