<template>
<Header></Header>
<div class="divisoria" ></div>
<div class="Tudo">
    <div class="menuesquerdo" >
        <h2> <span class="h2fake" > Olá, {{ usuario.name }}</span> 👋 </h2>
        <router-link to="/dados"> <button>Meus dados</button></router-link>
        <router-link to="/carrinho"> <button>Carrinho</button></router-link>
        <router-link to="/favoritos"> <button>Favoritos</button></router-link>
        <router-link to="/pedidos"> <button>Pedidos</button></router-link>
        <router-link to="/enderecos"> <button>Endereços</button></router-link>
        <router-link to="/cupons"> <button>Cupons</button></router-link>
        <div class="admin" v-if="userRole === 'ADMIN' || userRole === 'MODERATOR'">
        <h3>ADMIN</h3>
        <router-link to="/ADMmoderadores"> <button>Gerenciar Moderadores</button></router-link>
        <router-link to="/ADMcategorias"> <button>Gerenciar categorias</button></router-link>
        <router-link to="/ADMprodutos"> <button>Gerenciar produtos</button></router-link>
        <router-link to="/ADMpedidos"> <button>Gerenciar Pedidos</button></router-link>
        <router-link to="/ADMcupons"> <button>Gerenciar Cupons</button></router-link>
        <router-link to="/ADMdescontos"> <button>Gerenciar Descontos</button></router-link>
        </div>
    </div>
    <div class="menudireito" >
        <router-view></router-view>
    </div>
</div>
<Footer></Footer>
</template>

<script setup>
import Header from '../components/Headercomponent.vue'
import Footer from '../components/Footercomponent.vue'
import { ref, onMounted } from 'vue'
import api from '../services/api'

const usuario = ref({})
const userRole = ref(null)

onMounted(async () => {
    try {
        const token = localStorage.getItem('token')
        if (token) {
            api.defaults.headers.common['Authorization'] = `Bearer ${token}`
            const { data } = await api.get('/users/me')
            usuario.value = data
            userRole.value = data.role
        }
    } catch (e) {
        userRole.value = null
    }
})
</script>

<style scoped>

.divisoria{
    width: 100%;
    height: 1px;
    background-color: #06080afa;
}

.Tudo{
    display: flex;
    justify-content: center;
    width: 100%;
    height: 100vh;
    padding: 20px;
    box-sizing: border-box;
    overflow-x: hidden;
}

.menuesquerdo{
    width: 20vw;
    max-width: 400px;
    min-width: 280px;
    height: 100%;
    background-color: #06080afa;
    border: px solid white;
    z-index: 10;
    border-radius: 12px 0 0 12px;
    flex-shrink: 0;
}

.menuesquerdo h2 {
    color: white;
    text-align: center;
    padding-top: 20px;
    font-size: clamp(1.2rem, 2vw, 1.8rem);
}

.h2fake {
    text-decoration: underline;
}

.menuesquerdo h3{
    color: #ffffff;
    font-size: clamp(1.3rem, 2.2vw, 2rem);
    font-weight: bold;
    text-align: center;
    margin-top: 20px;
    text-decoration: underline;
}

.menuesquerdo button{
    width: 100%;
    height: clamp(50px, 6vh, 80px);
    color: #ffffff;
    font-weight: bold;
    font-size: clamp(0.9rem, 1.5vw, 1.3rem);
}



.menuesquerdo button:hover{
    transition: 0.1s;
    background-color: #eeeeee;
    color: #000000;
}


.menudireito{
    width: calc(100% - 20vw);
    max-width: calc(100% - 400px);
    min-width: calc(100% - 280px);
    height: 100%;
    background-color: #ffffff;
    border: 1px solid black;
    overflow-x: hidden;
    min-width: 0;
    flex: 1;
}

.admin button{
    color: #00b7ff;
}

/* Responsividade para telas grandes (TVs e monitores) */
@media (min-width: 1920px) {
    .menuesquerdo {
        width: 18vw;
        max-width: 500px;
        min-width: 350px;
    }
    
    .menudireito {
        width: calc(100% - 18vw);
        max-width: calc(100% - 500px);
        min-width: calc(100% - 350px);
    }
    
    .menuesquerdo h2 {
        font-size: clamp(1.5rem, 2.5vw, 2.2rem);
    }
    
    .menuesquerdo h3 {
        font-size: clamp(1.6rem, 2.8vw, 2.5rem);
    }
    
    .menuesquerdo button {
        height: clamp(70px, 7vh, 100px);
        font-size: clamp(1.1rem, 1.8vw, 1.6rem);
    }
}

@media (min-width: 2560px) {
    .menuesquerdo {
        width: 16vw;
        max-width: 600px;
        min-width: 400px;
    }
    
    .menudireito {
        width: calc(100% - 16vw);
        max-width: calc(100% - 600px);
        min-width: calc(100% - 400px);
    }
    
    .menuesquerdo h2 {
        font-size: clamp(1.8rem, 3vw, 2.8rem);
    }
    
    .menuesquerdo h3 {
        font-size: clamp(2rem, 3.5vw, 3.2rem);
    }
    
    .menuesquerdo button {
        height: clamp(80px, 8vh, 120px);
        font-size: clamp(1.3rem, 2.2vw, 2rem);
    }
}

/* Responsividade */
@media (max-width: 768px) {
    .Tudo {
        flex-direction: row;
        height: 100vh;
        padding: 10px;
    }
    
    .menuesquerdo {
        width: 25vw;
        max-width: 200px;
        min-width: 180px;
        height: 100%;
        border-radius: 12px 0 0 12px;
        margin-bottom: 0;
        padding: 15px;
        box-sizing: border-box;
    }
    
    .menudireito {
        width: calc(100% - 25vw);
        max-width: calc(100% - 200px);
        min-width: calc(100% - 180px);
        height: 100%;
        min-height: auto;
    }
    
    .menuesquerdo button {
        height: clamp(45px, 5vh, 60px);
        font-size: clamp(0.8rem, 1.2vw, 1rem);
    }
}

@media (max-width: 480px) {
    .Tudo {
        padding: 5px;
    }
    
    .menuesquerdo {
        width: 30vw;
        max-width: 150px;
        min-width: 140px;
        padding: 10px;
        box-sizing: border-box;
    }
    
    .menudireito {
        width: calc(100% - 30vw);
        max-width: calc(100% - 150px);
        min-width: calc(100% - 140px);
    }
    
    .menuesquerdo button {
        height: clamp(40px, 4vh, 50px);
        font-size: clamp(0.7rem, 1vw, 0.9rem);
    }
}

</style>