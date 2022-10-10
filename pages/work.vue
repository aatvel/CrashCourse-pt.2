<template>
        <div class="bg-black h-screen px-12 py-12">

            <h2 class="text-white text-6xl font-sans italic  opacity-0" 
            ref="title" 
            style="transform: translateY(30px)" >MY WORK</h2>

            <div class="flex space-x-6">
                <div 
                ref="projects" v-for="project in projects"                
                class="w-full opacity-0 " 
                style="transform: translateX(30px)">

                <nuxt-link to="/">

                    <img :src="project.image.url"
                     alt="Planet" 
                     style="height: 70vh;"
                     class="object-cover"
                     >
                    <p class="text-white font-sans text-sm">{{project.title}}</p>
                
                </nuxt-link>
                </div>
               
              
        </div>
        </div>
</template>

<script>
import planetImg from '~/static/pic.jpg';
import sunImg from '~/static/sun.jpg';
import planImg from '~/static/plan.jpg';

import gsap from 'gsap';

export default {
    data(){
        return{
            projects: [
            
            {
                image: {
                    url: sunImg
                },
                title: 'SUN'
            },
            {
                image: {
                    url: planetImg
                },
                title: 'MOON'
            },
            {
                image: {
                    url: planImg
                },
                title: 'EARTH'
            },
            {
                image: {
                    url: sunImg
                },
                title: 'SUN'
            },
            {
                image: {
                    url: planetImg
                },
                title: 'MOON'
            },
            {
                image: {
                    url: planImg
                },
                title: 'EARTH'
            }
        ]
        }
    },
    mounted(){
        gsap.to(this.$refs.title, {
            opacity: 1,
            duration: 2,
            y:0,
            ease: 'expo'
        })

        gsap.to(this.$refs.projects, {
            opacity: 1,
            duration: 2,
            stagger: 0.2,
            x: 0,
            ease: 'expo',
            delay: 0.4
        })
    }
}
</script>