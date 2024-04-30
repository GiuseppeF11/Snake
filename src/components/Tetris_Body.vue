<script>

export default {
    data() {
        return {

        };
    },
    methods: {

    }
}

/* TETRIS */
document.addEventListener('DOMContentLoaded', function() {
    console.log('ciao'); //test
    const screen = document.getElementById('screen');       //Selezioniamo il campo di gioco
    const ctx = screen.getContext('2d');       //Consente di disegnare sullo screen , il canvas in questo caso

    //SCHERMO
    const SCALE_F = 20;
    const COLOR1 = '#00AA8D';
    const COLOR2 = '#00BF9A';

    //GRIGLIA
    ctx.scale(SCALE_F,SCALE_F)
    for (let i = 0; i < screen.width / SCALE_F; i++) {
        ctx.fillStyle = i % 2 === 0 ? COLOR1 : COLOR2;
        ctx.fillRect( i , 0 , 1 , screen.height / SCALE_F );
        console.log(i);
    }

    //BLOCCHI
    let element =  [
        [ 0 , 1 , 0 , 0],
        [ 1 , 1 , 1 , 0],
        [ 0 , 0 , 0 , 0],
        [ 0 , 0 , 0 , 0],
    ]

    ctx.fillStyle = 'green';

    function drawElement(el) {
        element.forEach((row, y) => {
            row.forEach((v , x) => {
                console.log(v , x , y);
                if (v != 0) {
                    ctx.fillRect(x , y , 1 , 1);
                }
            })
        })
    }

    drawElement(element);
});
</script>

<template>
    <canvas id="screen" class="screen m-auto border-4 ">
        ciao
    </canvas>
</template>
 


<style lang="scss" scoped>
.screen {
    width: 400px;
    height: 400px;
}


</style>