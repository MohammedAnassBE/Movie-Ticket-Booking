<template>
    <div>
        <div class="mt-11 flex flex-row items-center justify-between p-3 shadow-2xl">
            <h1 v-for="datas in movieList.data" :key="datas">
                <div class="mt-11 flex flex-col items-center justify-between p-5 shadow-2xl">
                    <div>
                        <h2 class="text-2xl">Director-{{ datas.director}}</h2>
                    </div>
                    <div>
                        <img :src="datas.poster" class="w-[260px] h-[260px] mt-4" alt="">
                    </div>
                    <div> 
                        <h2 class="text-2xl font-bold mt-4">{{ datas.movie_name}}</h2>
                    </div>
                    <router-link :to="{ name : 'MovieDetails' , params:{movieName : datas.name}}">        
                        <button class="bg-blue-600 w-40 rounded mt-4">Book</button>
                    </router-link>
                </div>
            </h1>
        </div>    
    </div>            
</template>

<script setup> 
import { createListResource } from "frappe-ui";
import { RouterLink } from "vue-router";

const movieList = createListResource({
    doctype : 'Movie',
    fields: ['name','director','movie_name','release_date','synopsis','poster'],
    auto: true,
})
</script>

<style scoped>

</style>