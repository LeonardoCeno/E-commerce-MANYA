<template>
<div class="tudo" >
    <div class="dados-container">
        <input type="file" ref="fileInput" accept="image/*" style="display:none" @change="onFileChange" />
        <img :src="userImageUrl" alt="" class="foto-usuario" @click="triggerFileInput" title="Clique para alterar a foto" />
        <div v-if="carregando" class="carregando">Carregando dados...</div>
        <div v-else-if="erro" class="erro">{{ erro }}</div>
        <div v-else>
            <div class="informacoes" ref="informacoesRef">
                <div class="info-item">
                    <img src="/src/components/img/editando.png" alt="editar" class="edit-icon" @click="editarNome" />
                    <template v-if="editandoNome">
                        <input v-model="novoNome" ref="nomeInputRef" type="text" class="info-input" autofocus />
                    </template>
                    <template v-else>
                        <p class="info-label"><strong>Nome:</strong> {{ usuario.name }}</p>
                    </template>
                </div>
                <div class="info-item">
                    <img src="/src/components/img/editando.png" alt="editar" class="edit-icon" @click="editarEmail" />
                    <template v-if="editandoEmail">
                        <input v-model="novoEmail" ref="emailInputRef" type="email" class="info-input" autofocus />
                    </template>
                    <template v-else>
                        <p class="info-label"><strong>Email:</strong> {{ usuario.email }}</p>
                    </template>
                </div>
                <div class="info-item">
                    <img src="/src/components/img/editando.png" alt="editar" style="opacity: 0;" class="edit-icon" @click="editarSenha" />
                    <template v-if="editandoSenha">
                        <input v-model="novaSenha" ref="senhaInputRef" @blur="salvarSenha" @keyup.enter="salvarSenha" type="password" class="info-input" placeholder="Nova senha" autofocus />
                    </template>
                    <template v-else>
                        <p class="info-label"><strong>Senha:</strong> ********</p>
                    </template>
                </div>
                <button v-if="editandoNome || editandoEmail" class="confirmar-btn" @click="confirmarEdicao">Confirmar alterações</button>
                <button v-if="editandoNome || editandoEmail" class="cancelar-btn" @click="cancelarEdicao">Cancelar</button>
            </div>
        </div>
    </div>
</div>
</template>

<script setup>
import { ref, onMounted, computed, onBeforeUnmount, nextTick } from 'vue'
import api, { getUsuario } from '../services/api'

const usuario = ref({})
const carregando = ref(true)
const erro = ref('')
const fileInput = ref(null)
const Inputfile = ref(null)
const wallpaperStyle = ref({})

const BASE_IMAGE_URL = 'http://35.196.79.227:8000'

const userImageUrl = computed(() => {
    if (!usuario.value.image_path) return '/placeholder-image.jpg'
    return BASE_IMAGE_URL + usuario.value.image_path
})

const editandoNome = ref(false)
const editandoEmail = ref(false)
const editandoSenha = ref(false)
const novoNome = ref('')
const novoEmail = ref('')
const novaSenha = ref('')

const nomeInputRef = ref(null)
const emailInputRef = ref(null)
const senhaInputRef = ref(null)

function useClickOutside(targetRef, callback) {
    function handler(event) {
        if (targetRef.value && !targetRef.value.contains(event.target)) {
            callback()
        }
    }
    onMounted(() => {
        document.addEventListener('mousedown', handler)
    })
    onBeforeUnmount(() => {
        document.removeEventListener('mousedown', handler)
    })
}

useClickOutside(senhaInputRef, () => { if (editandoSenha.value) editandoSenha.value = false })

const informacoesRef = ref(null)

function cancelarEdicao() {
    if (editandoNome.value) {
        novoNome.value = usuario.value.name
        editandoNome.value = false
    }
    if (editandoEmail.value) {
        novoEmail.value = usuario.value.email
        editandoEmail.value = false
    }
}

function useClickOutsideDiv(targetRef, callback) {
    function handler(event) {
        if (targetRef.value && !targetRef.value.contains(event.target)) {
            callback()
        }
    }
    onMounted(() => {
        document.addEventListener('mousedown', handler)
    })
    onBeforeUnmount(() => {
        document.removeEventListener('mousedown', handler)
    })
}

useClickOutsideDiv(informacoesRef, cancelarEdicao)

onMounted(async () => {
    await carregarUsuario()
    const savedWallpaper = localStorage.getItem('wallpaperBg')
    if (savedWallpaper) {
        wallpaperStyle.value = {
            backgroundImage: `url('${savedWallpaper}')`,
            backgroundSize: 'cover',
            backgroundPosition: 'center',
            backgroundRepeat: 'no-repeat'
        }
    }
})

async function carregarUsuario() {
    try {
        usuario.value = await getUsuario()
        novoNome.value = usuario.value.name
        novoEmail.value = usuario.value.email
        novaSenha.value = ''
    } catch (e) {
        erro.value = 'Erro ao carregar dados do usuário.'
    } finally {
        carregando.value = false
    }
}

function triggerFileInput() {
    if (fileInput.value) fileInput.value.click()
}

async function onFileChange(event) {
    const file = event.target.files[0]
    if (!file) return
    const formData = new FormData()
    formData.append('image', file)
    try {
        const response = await api.put('/users/image', formData, {
            headers: {
                'Content-Type': 'multipart/form-data'
            }
        })
        usuario.value = response.data
    } catch (e) {
        erro.value = 'Erro ao atualizar imagem de perfil.'
    }
}

function editarNome() {
    editandoNome.value = true
}

function editarEmail() {
    editandoEmail.value = true
}

function editarSenha() {
    editandoSenha.value = true
    novaSenha.value = ''
}

async function confirmarEdicao() {
    let alterou = false
    try {
        if (editandoNome.value && novoNome.value && novoNome.value !== usuario.value.name) {
            alterou = true
        }
        if (editandoEmail.value && novoEmail.value && novoEmail.value !== usuario.value.email) {
            alterou = true
        }
        if (alterou) {
            const response = await api.put('/users/me', { name: novoNome.value, email: novoEmail.value })
            usuario.value = response.data
        }
        editandoNome.value = false
        editandoEmail.value = false
    } catch (e) {
        erro.value = 'Erro ao atualizar dados.'
    }
}

</script>

<style scoped>

.tudo {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    height: 100%;
    background-color: #f8f9fa;
    padding: 20px;
    box-sizing: border-box;
}

.dados-container {
    margin-top: 1vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    max-width: 40%;
    background: #fff;
    padding: 40px 24px;
    font-family: 'Inter', Arial, sans-serif;
    border-radius: 12px;
    box-shadow: 0 4px 16px rgba(0,0,0,0.08);
    box-sizing: border-box;
    width: 100%;
    min-width: 500px;
    min-height: 500px;
}

h2 {
    color: #000000;
    margin-bottom: 24px;
    text-align: center;
    font-size: 3vw;
}
p {
    font-size: 2vw;
    margin-bottom: 12px;
}

.carregando {
    color: #666;
    font-size: 1rem;
    text-align: center;
    padding: 20px;
}

.erro {
    color: #e11d48;
    text-align: center;
    margin-top: 20px;
    background: #fee;
    padding: 12px;
    border-radius: 8px;
    border: 1px solid #feb2b2;
}

.informacoes {
    margin-top: 70px;
    display: flex;
    flex-direction: column;
    gap: 12px;
    width: 100%;
}

.foto-usuario {
    width: 35vw;
    height: 35vw;
    min-width: 180px;
    min-height: 180px;
    max-width: 350px;
    max-height: 350px;
    border-radius: 50%;
    border: 3px solid #e1e5e9;
    margin: 18px 0 32px 0;
    cursor: pointer;
    transition: all 0.3s ease;
    z-index: 2;
    object-fit: cover;
    background-repeat: no-repeat;
    background-position: center;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.foto-usuario:hover {
    transition: 0.1s;
    background-color: rgb(196, 196, 196);
    background-image: url('../components/img/baixar.png');
    background-size: 15%;
    opacity: 0.9;
    filter: brightness(80%);
    transform: scale(1.02);
}

.info-item {
    display: flex;
    align-items: center;
    margin-bottom: 8px;
    flex-wrap: nowrap;
    gap: 6px;
    background: #f8f9fa;
    padding: 12px;
    border-radius: 8px;
    border: 1px solid #e9ecef;
    transition: all 0.2s ease;
    width: 100%;
    box-sizing: border-box;
}

.info-item:hover {
    background: #f1f3f4;
    box-shadow: 0 2px 8px rgba(0,0,0,0.06);
}

.info-label {
    margin: 0;
    user-select: none;
    transition: color 0.2s;
    color: #333;
    text-decoration: none;
    font-size: 160%;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    font-weight: 500;
    flex-shrink: 0;
    min-width: 0;
}

.edit-icon {
    width: 20px;
    height: 20px;
    margin-right: 6px;
    cursor: pointer;
    flex-shrink: 0;
    transition: transform 0.2s ease;
}

.edit-icon:hover {
    transform: scale(1.1);
}

.info-input {
    font-size: 1.6vw;
    min-width: 0;
    flex: 1 1 0%;
    padding: 6px 10px;
    border: 2px solid #e1e5e9;
    border-radius: 6px;
    background: #fff;
    transition: border 0.2s ease;
    box-sizing: border-box;
    max-width: 100%;
}

.info-input:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.confirmar-btn {
    margin-top: 16px;
    padding: 10px 20px;
    background: #030a11f5;
    color: #fff;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 2px 8px rgba(3, 10, 17, 0.2);
}

.confirmar-btn:hover {
    background: #02060ac2;
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(3, 10, 17, 0.3);
}
.cancelar-btn {
    margin-top: 8px;
    padding: 10px 20px;
    background: #e0e0e0;
    color: #333;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}
.cancelar-btn:hover {
    background: #cccccc;
    color: #111;
}

@media (max-width: 1000px) {
    .dados-container {
        max-width: 60%;
        min-width: 280px;
        min-height: 450px;
    }
    
    .info-label, .info-input {
        font-size: 3.2vw;
    }
    p {
    font-size: 3vw;
    margin-bottom: 12px;
}
    .wallpaper {
        height: 275px;
    }
    .informacoes {
        margin-top: 50px;
    }
}

@media (max-width: 768px) {
    .info-label, .info-input {
        font-size: 1.7vw;
    }
}

@media (max-width: 580px) {
    .dados-container {
        max-width: 80%;
        padding: 24px 14px;
        min-width: 260px;
        min-height: 400px;
    }
    
    .info-label, .info-input {
        font-size: 2.3vw;
    }
    .wallpaper {
        width: 70vw;
        height: 205px;
    }
    .informacoes {
        margin-top: 25px;
        gap: 8px;
    }
    
    .info-item {
        padding: 10px;
        margin-bottom: 6px;
    }
    
    .confirmar-btn {
        padding: 8px 16px;
        font-size: 0.9rem;
    }
    
    .foto-usuario {
        width: 160px;
        height: 160px;
        min-width: 140px;
        min-height: 140px;
        max-width: 180px;
        max-height: 180px;
    }
    
    .edit-icon {
        width: 18px;
        height: 18px;
    }
}

</style>