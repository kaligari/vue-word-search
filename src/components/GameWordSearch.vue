<template>
        <div class="columns" id="game-words">
            <div class="column"></div>
            <div class="column is-7">
                <div class="letters">
                    <div class="row" v-for="row in letters">
                        <p v-for="col in row" :class="[{'word':col.reserved},{'clicked':col.clicked}]" @click="selectLetter(col.row,col.col)">{{ col.letter }}</p>
                    </div>
                </div>
            </div>
            <div class="column has-text-centered-mobile">
                <h3>Find words:</h3>
                <ul>
                    <li v-for="word in wordsArray" :class="{'done':( typeof word.letters != 'undefined' && word.letters instanceof Array )}">{{ word.word }}</li>
                </ul>
                <h3>Time:</h3>
                {{ time }} s.
            </div>
            <transition name="fade">
                <div class="modal is-active" v-if="modal">
                    <div class="modal-background"></div>
                    <div class="modal-content">
                        <div class="box has-text-centered">
                            <p>Your time is {{ time }} s.</p><br />
                            <br />
                            <button type="button" class="button is-primary" @click="modal = false">Close</button>
                        </div>
                    </div>
                    <button class="modal-close is-large" aria-label="close" @click="modal = false"></button>
                </div>
            </transition>
        </div>
</template>

<script>
export default {
    name: 'GameWords',    
    data: function(){
        return{
            time: 0,
            timeInterval: null,
            total: 0,
            modal: false,
            letters: [],
            wordsArray: []
        }
    },
    props: {
        mainUrl: {
            type: String,
            required: true,
        },
        words:{
            type: Array,
        },
        size: {
            default:7,
        }
    },
    methods: {
        selectLetter(indexX,indexY){
            // sprawdz czy ta litera jest zwiazana z jakims slowem
            if(this.letters[indexX][indexY].reserved == true){
                // jesli tak to oznacz bool clicked zeby dodalo klase
                this.letters[indexX][indexY].clicked = true;
                // w petli przejzyj wszystkie wyrazy
                for(let w = 0; w < this.wordsArray.length; w++){
                    // a konkretnie litery ich wyrazow jesli istnieja
                    if( typeof this.wordsArray[w].letters != 'undefined' && this.wordsArray[w].letters instanceof Array ){
                        // czyli kazda litere
                        for(let l = 0; l < this.wordsArray[w].letters.length; l++){
                            // jesli ID litery jest takie jak id kliknietej litery
                            if(this.wordsArray[w].letters[l] == this.letters[indexX][indexY].id){
                                // to usun ja z tablicy liter danego wyrazu
                                this.wordsArray[w].letters.splice(l,1);
                            }
                        }
                        // jesli aktualny wyraz jest juz pusty czyli tablica liter jest pusta
                        if(this.wordsArray[w].letters.length == 0){
                            // to usun tablice
                            delete this.wordsArray[w].letters;
                            // i zwiększ zmienn od wszystkich wyrazów
                            this.total++;
                        }

                    }
                }
            }
            if(this.total == this.words.length){
                // zawibruj jesli telefon
                // window.navigator.vibrate(800);
                clearInterval(this.timeInterval);
                setTimeout(()=>{
                    this.modal = true;
                },100);
            }
        },
        putWord(word){
            // tymczasowe zmienne do umieszczenia słów
            let collision, start, end, direction, x, pos1, pos2, secondPosition, col;
            // zaczynamy od 0 kolizji
            col = 0;
            // pętla która sprawdza czy w losowym miejscu będzie można wsadzić wyraz
            // jeśli będzie przeszkoda czyli zarezerwowane już pole z litera i to nie będzie
            // litera która pasuje, to rób pętle dalej
            do{
                // na starcie nie ma kolizji
                collision = false;
                // ustal dlugosc
                let length = word.length;
                // ustal kierunek: true poziomo, false pionowo
                direction = (Math.floor(Math.random() * 2)==0) ? true : false;
                // ustal mozliwe punkt startu
                let startEnd = this.size - length;
                // wybierz randomow liczbę z przedziału
                start = Math.floor(Math.random() * startEnd);
                // ustal koniec wyrazu
                end = start+length;
                // ustal drugi wymiar
                secondPosition = Math.floor(Math.random() * this.size);
                // sprawdz czy nie ma kolizji
                for(x = start; x < end; x++){
                    // okresl czy pion czy poziom
                    pos1 = direction ? x : secondPosition;
                    pos2 = direction ? secondPosition : x;
                    // warunek sprawdza czy pole zajete i czy litera jest inna, jesli oba spełnione to oznacza że kolizja
                    if(this.letters[pos1][pos2].reserved == true && this.letters[pos1][pos2].letter != word[x - start]){
                        collision = true;
                        col++;
                    }
                }
            } while (collision == true && col < 20);
            // jeśli próbujemy już ponad 20 razy to znaczy że nie ma gdzie wsadzić wyrazu
            if(col >= 20){
                // i zwracamy bład
                console.error('Initialization error');
            }
            // jesli jest ok to wsadz wyrazy do nowej tablicy słów
            this.wordsArray.push({word:word,letters:[]});
            // wsadz litery
            for(x = start; x < end; x++){
                // okresl czy pion czy poziom
                pos1 = direction ? x : secondPosition;
                pos2 = direction ? secondPosition : x;
                // przypisz wartosci
                this.letters[pos1][pos2].letter = word[x - start];
                this.letters[pos1][pos2].reserved = true;
                // wsadz litery do tablicy slow
                this.wordsArray[this.wordsArray.length-1].letters.push(this.letters[pos1][pos2].id)
            }
        }
    },
    created(){
        // lista możliwych znaków
        let possible = 'abcdefghijklmnopqrstuvwxyz';
        // zainicjuj tablicę
        this.letters = [];
        // pętla wierszy
        for(let x = 0; x < this.size; x++){
            // zainicjuj tymczasow tablicę elementów wiersza
            let row = [];
            // pętla kolumn
            for(let y = 0; y < this.size; y++){
                // wypełnij tablicę wiersza danymi o literce
                row.push({
                    letter : possible.charAt(Math.floor(Math.random() * possible.length)),
                    id: ((x*this.size)+y),
                    reserved : false,
                    clicked : false,
                    row: x,
                    col: y
                });
            }
            // dodaj tymczasowa listę wiersza do tablicy wszystkich liter
            this.letters.push(row);
        }
        // dodaj wyrazy do tablicy wyrazów
        for(let w = 0; w < this.words.length; w++){
           this.putWord(this.words[w]);
        }
        // rozpocznij liczenie czasu
        this.timeInterval = setInterval(()=>{
            this.time++;
        },1000);
    }
}
</script>

<style lang="less">
#game-words{
    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
    }
    .fade-enter, .fade-leave-to{
        opacity: 0;
    }
    ul{
        margin-left: 0;
        list-style-position: inside;

        li{
            text-decoration: line-through;
            &.done{
                text-decoration: none;
            }
        }
    }
    .letters{
        display: block;
        margin: auto;
        text-align: center;
        p{
            display: inline-block;
            margin: 0;
            width: 40px;
            height: 40px;
            line-height: 40px;
            text-align: center;
            vertical-align: middle;
            border: 1px solid #000;
            cursor:pointer;
            text-transform: uppercase;
            font-size: 18px;
            &.word{
                // background-color: pink;
            }
            &.clicked{
                background-color: lightgreen;
            }
        }
    }
}

// RWD
@media screen and (max-width: 768px) {
    #game-words{
        .fade-enter-active, .fade-leave-active {
            transition: opacity .5s
        }
        .fade-enter, .fade-leave-to{
            opacity: 0
        }
        ul{
            li{
                text-decoration: line-through;
                &.done{
                    text-decoration: none;
                }
            }
        }
        .letters{

            p{
                width: 32px;
                height: 32px;
                line-height: 32px;
            }
        }
    }
}


</style>
