## Замыканні і спасылкі

Адна з найбольш магутных магчымасцяў JavaScript — магчымасць ствараць *замыканні*.
Зона бачнасці замыканняў **заўсёды** мае доступ да знешняй зоны бачнасці, у якой
замыканне было аб'яўлена. З той прычыны, што ў JavaScript адзіны механізм працы
з зонай бачнасці — гэта [зоны бачнасці функцыі](#function.scopes), усе функцыі
выступаюць у якасці замыканняў.

### Эмуляцыя прыватных пераменных

    function Counter(start) {
        var count = start;
        return {
            increment: function() {
                count++;
            },

            get: function() {
                return count;
            }
        }
    }

    var foo = Counter(4);
    foo.increment();
    foo.get(); // 5

Тут `Counter` вяртае **два** замыканні: функцыю `increment` і функцыю `get`.
Абедзьве функцыі маюць **спасылку** на зону бачнасці `Counter` і таму заўсёды
маюць доступ да пераменнай `count`, што была аб'яўлена ў гэтай зоне бачнасці.

### Якім чынам гэта працуе

З той прычыны, што ў JavaScript немагчыма спасылацца або прысвойваць зоны бачнасці,
**немагчыма** атрымаць доступ да пераменнай `count` звонку. Адзіны спосаб
узаемадзейнічаць з ім — выкарыстоўваць два замыканні.

    var foo = new Counter(4);
    foo.hack = function() {
        count = 1337;
    };

Вышэйпрыведзены код **не** памяняе значэнне пераменнай `count` у зоне бачнасці
`Counter`, бо `foo.hack` не быў аб'яўлены у **гэтай** зоне бачнасці. Замест гэтага
ён створыць або перазапіша *глабальную* пераменную `count`.

### Замыканні ўнутры цыклаў

Частая памылка - выкарыстанне замыканняў унутры цыклаў, як быццам бы яны капіруюць
значэнне пераменнай індэксу цыкла.

    for(var i = 0; i < 10; i++) {
        setTimeout(function() {
            console.log(i);  
        }, 1000);
    }

Вышэйпрыведзены код **не** выведзе нумары ад `0` да `9`, ён проста выведзе
нумар `10` дзесяць разоў.

*Ананімная* функцыя захоўвае **спасылку** на `i`. У той час, калі функцыя `console.log`
выклікаецца, `цыкл for` ужо адпрацаваў, а значэнне `i` ўжо стала `10`.

Каб атрымаць пажаданыя паводзіны, неабходна стварыць **копію** значэння `i`.

### Як абыйсці праблемы спасылкі

Каб стварыць копію значэння пераменнай індэкса цыкла, лепшы спосаб — стварэнне
[ананімнай абгорткі](#function.scopes).

    for(var i = 0; i < 10; i++) {
        (function(e) {
            setTimeout(function() {
                console.log(e);  
            }, 1000);
        })(i);
    }

Знешняя ананімная функцыя выконваецца імгненна з `i` ў якасці першага аргумента
і атрымае копію **значэння** `i` ў якасці параметра `e`.

Ананімная функцыя, што перадаецца метаду `setTimeout`, цяпер мае спасылку на
`e`, чыё значэнне **не** мяняецца на працягу цыкла.

Яшчэ адзін спосаб атрымаць такі вынік — вяртаць функцыю з ананімнай абгорткі,
што будзе паводзіць сябе такім жа чынам, як і папярэдні прыклад.

    for(var i = 0; i < 10; i++) {
        setTimeout((function(e) {
            return function() {
                console.log(e);
            }
        })(i), 1000)
    }

Яшчэ адзін папулярны спосаб дасягнуць гэтага — дадаць яшчэ адзін аргумент выкліку
функцыі `setTimeout`, якая перадасць агрумент функцыі зваротнага выкліку.

    for(var i = 0; i < 10; i++) {
        setTimeout(function(e) {
            console.log(e);  
        }, 1000, i);
    }

Некаторыя старыя асяродкі JS (Internet Explorer 9 і ніжэй) не падтрымліваюць
гэтую магчымасць.

Таксама магчыма выканаць гэта выкарыстоўваючы `.bind`, якая можа звязаць
`this` і аргументы функцыі. Ніжэйпрыведзены прыклад працуе як папярэднія.

    for(var i = 0; i < 10; i++) {
        setTimeout(console.log.bind(console, i), 1000);
    }