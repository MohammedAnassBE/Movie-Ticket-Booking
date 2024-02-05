<template>
     <div> 
        <div class="mt-11 flex flex-row items-center justify-between">
            <h1 class="font-bold text-gray-800 text-[32px]">{{ movieDoc.movie_name }}</h1>
            <div class="'flex flex-col space-y-3">
                <h2 class="text-bold text-gray-800 uppercase text-xl">Director</h2>
                <h2 class="text-bold text-gray-600 text-xl">{{ movieDoc.director }}</h2>
            </div>
            <div class="'flex flex-col space-y-3">
                <h2 class="text-bold text-gray-800 uppercase text-xl">Release Date</h2>
                <h2 class="text-bold text-gray-600 text-xl">{{ movieDoc.release_date }}</h2>
            </div>
        </div>
        <div v-if="page === 0">    
            <div class=" max-w-full flex flex-col items-center justify-center p-2 mx-8 shadow-2xl bg-white mt-7">
                <div>
                    <img :src="movieDoc.poster" class="w-[260px] h-[260px]" alt="">
                </div>        
            </div>
            <div class="max-w-full flex flex-col items-center justify-center m-7">
                <button @click="page=1" class="bg-blue-600 w-40 rounded" >Book Tickets</button>
            </div>
            <div class="flex flex-col space-y-3 mt-15 shadow-2xl p-2">
                <h2 class="text-gray-600 text-base font-bold uppercase">Synopsis</h2>
                <p>{{ movieDoc.synopsis }}</p>
            </div>
            <div class="flex flex-col space-y-3 mt-15 shadow-2xl p-2">
                <router-link :to="'/'"><button class="bg-blue-600 w-40 rounded" >Back</button></router-link>
            </div>
        </div>
        <div v-if="page === 1" class="mt-11 flex flex-col">
            <h2 class="text-bold text-gray-800 text-xl">How many seats</h2>
            <div class="flex flex-col w-full space-y-5">
                <button :style="{ backgroundColor: bookingDetails.numberOfSeats === n ? 'lightblue' : 'white' }" @click="seatCount(n)" class='p-2 m-1 shadow-2xl bg-white w-full rounded' v-for="n in 10" :key="n">{{n}}</button>
            </div>
            <div class="max-w-full flex flex-row items-center justify-center m-7">
                <button @click="page=0" class="bg-blue-300 w-40 rounded" >Back</button>
                <button @click="page=2" class="bg-blue-600 w-40 rounded" >Next</button>
            </div>
        </div>
        <div v-if="page === 2" class="mt-11 flex flex-col space-y-4">
            <div class="flex flex-col space-y-4">
                <h2 class="text-bold text-gray-800 text-xl">Date</h2>
                <Input type="date" :value='today' v-model="bookingDetails.movieDate"/>
            </div>
            <div class="flex flex-col space-y-4 p-4 rounded">
                <h2 class="text-bold text-gray-800 text-xl p-2">Select Cinema & Show</h2>
                <div v-for="theaters in Object.keys(theaterResource.data)" :key="theaters" class=" bg-white shadow-2xl">
                    <h2 class="p-2">{{theaters}}</h2>
                    <div class="flex flex-row p-2">
                        <Button v-for="details in theaterResource.data[theaters]" :key="details" @click="setShowTime(details.show_time,theaters)" :style="{ backgroundColor: bookingDetails.theaterName === theaters && bookingDetails.showTime === details.show_time ? 'lightblue' : 'white' }">{{details.show_time}}</Button>
                    </div>
                </div>
            </div>
            <div class="max-w-full flex flex-row items-center justify-center m-7">
                <button @click="page=1" class="bg-blue-300 w-40 rounded" >Back</button>
                <button @click="page = bookingDetails.theaterName === '' ? 2 : 3"
 class="bg-blue-600 w-40 rounded" >Next</button>
            </div>
        </div>    
        <div v-if="page === 3">
            <h2 class="text-bold text-gray-800 text-xl mt-5">{{bookingDetails.theaterName}}   {{bookingDetails.showTime}}</h2>
            <h2 class="text-bold text-gray-800 text-xl mt-5">Select Seats</h2>
            <div class="flex flex-col items-center justify-center mt-10">
                <div class="flex flex-row" v-for="row in Object.keys(seatStructure)" :key="row">   
                    <span @click=setSeat(row,seat[0])  v-for="seat in seatStructure[row]" :key="seat[0]" class="w-6 h-7 m-2 p-2 rounded" :class="seat[1] === 'Available' ? 'bg-blue-300' : seat[1]=== 'Booked' ? 'bg-red-300' : 'bg-gray-400' , identify() ? 'cursor-not-allowed' : 'cursor-pointer'" ></span>                    
                </div>
            </div>
            <div class="max-w-full flex flex-row items-center justify-center m-7">
                <button @click="page=2" class="bg-blue-300 w-40 rounded" >Back</button>
                <button @click="page = bookingDetails.selectedSeats.length === bookingDetails.numberOfSeats ? 4 : 3 , uplaodTicket()" class="bg-blue-600 w-40 rounded" >Next</button>
            </div>    
        </div>    
        <div v-if="page === 4" class="mt-11 flex flex-col items-center justify-center">
            <h2 class="text-bold text-gray-800 text-xl mt-5 ">{{bookingDetails.theaterName}}</h2>
            <h2 class="text-bold text-gray-800 text-xl mt-5 ">{{bookingDetails.showTime}}</h2>
            <h2 class="text-bold text-gray-800 text-xl mt-5 ">{{bookingDetails.numberOfSeats}} Seats</h2>
            <h2 class="text-bold text-gray-800 text-xl mt-5 ">Seat No : {{ JSON.stringify(bookingDetails.selectedSeats)}}</h2>
            <h2 class="text-[120px]">🍿</h2>
            <h2 class="text-bold text-gray-800 text-xl mt-5 ">Enjoy the Movie</h2>
            <div class="max-w-full flex flex-row items-center justify-center m-7">
                <router-link :to="'/'"><button class="bg-blue-600 w-40 rounded" >Home</button></router-link>
            </div>
        </div>   
    </div>
</template>

<script setup>
import {ref, reactive, computed} from "vue";
import { createDocumentResource, createListResource } from "frappe-ui";

const props=defineProps({
    movieName: {
        type: String,
        required: true,
    }
})

let movieResource =  createDocumentResource({
                        doctype: 'Movie',
                        name: props.movieName,
                        fields: ['name', 'director', 'release_date'],
                        auto: true,
                    });
const movieDoc = computed(() => movieResource.doc);

const theaterResource = createListResource({
    doctype: 'Theater Show',
    fields: ['theater', 'show_time', 'name'],
    auto: true,
    transform: (summa) => {
        const show={};
        for(const theaters of summa){
            if(!show[theaters.theater]){
                show[theaters.theater]=[];
            }
            show[theaters.theater].push(theaters);
        }
        return show;
    },
})
function getSeatStructure(alphabets,numbers){
    const structure={}
    for (const alphabet in alphabets){
        structure[alphabet]=[];
        for (const number in numbers){
            structure[alphabet].push([number,'Available']);
        }
    }
    return structure;
}
const seatStructure = reactive(getSeatStructure(['A','B','C','D','E'],[1,2,3,4,5,6]))
const today=new Date().toISOString().substr(0,10);
const page = ref(0);
const start = ref(0);
const bookingDetails = reactive({
    numberOfSeats : 1,
    movieDate : today,
    selectedSeats : [],
    seats : [],
    showTime : '',
    theaterName : '',
    movie : '',
})
const ticketBooking = createListResource({
    doctype : 'Theater Booking',
    insert: {},
})
function uplaodTicket(){

    ticketBooking.insert.submit({
        movie : bookingDetails.movie,
        theater : bookingDetails.theaterName,
        date : bookingDetails.movieDate,
        show : bookingDetails.showTime,
        tickets : bookingDetails.numberOfSeats,
        seats : JSON.stringify(bookingDetails.selectedSeats),   
    })
}
function seatCount(n){
    bookingDetails.numberOfSeats=n;
}
function setShowTime(datas, theater){
    bookingDetails.showTime=datas+"";
    bookingDetails.theaterName=theater;
}
function setSeat(row, number){
    if(identify()){
        return;
    }
    let alp='A'
    const seat=seatStructure[row].find((seat)=>(seat[0] === number));
    let newRowCode = alp.charCodeAt(0) + row;
    let newRow = String.fromCharCode((newRowCode % 65) % 26 + 65);
    seat[1]='Selected';
    bookingDetails.selectedSeats.push(`${newRow}${number}`);
}
function identify(){
    return bookingDetails.selectedSeats.length === bookingDetails.numberOfSeats
}
// function lastMethod(){
//     bookingDetails.movie='';
//     bookingDetails.movieDate=today;
//     bookingDetails.numberOfSeats=0;
//     bookingDetails.selectedSeats=[];
//     bookingDetails.showTime='';
//     bookingDetails.theaterName='';
//     for (const seats in Object.keys(seatStructure)){
//         for (const values in seatStructure[seats]){
//             for (const arr in values){
//                 console.log(arr[1]);
//             }    
//         }
//     }
    // console.log(seatStructure)
// }
</script>

<style scoped>

</style>