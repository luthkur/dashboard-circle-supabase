<template>
  <form class="form-widget" @submit.prevent="updateProfile">
    <!-- <Avatar v-model:path="circle_cut" @upload="updateProfile" /> -->
    <div>
      Hi PJ Circle {{circle_data.name}}
    </div>
    <div>
      Toggle which product types that your circle will showcase on the catalog
    </div>
    <div class="row">
      <div class="col-6-sm" v-for="toggle in listToggle" :key="toggle.value"> 
        <label >{{toggle.label}}</label>
        <input class="tgl tgl-light" :id="toggle.value" type="checkbox" v-model="circle_data[toggle.value]"/>
        <label class="tgl-btn" :for="toggle.value"> 
      </label>
      </div>
    </div>

    <div>
      Paste below complete link for your social media links and link to the circle shop. Example : https://www.facebook.com/Comifuro/ , https://twitter.com/comifuro
    </div>

    <div class="row" v-for="input in listInput" :key="input.value">
      <div class="col-12-sm" > 
        <label :for="input.value" >{{input.label}}</label>
        <input :id="input.value" type="text" v-model="circle_data[input.value]"/>
      </div>
    </div>
    
    
    <div>
      <input
        type="submit"
        class="button block primary"
        :value="loading ? 'Loading ...' : 'Update'"
        :disabled="loading"
      />
    </div>
    

    <div>
      <button class="button block" @click="signOut" :disabled="loading">
        Sign Out
      </button>
    </div>
  </form>
</template>

<script>
import { supabase } from "../supabase"
import { store } from "../store"
import { onMounted, ref } from "vue"
// import Avatar from "./Avatar.vue"

export default {
  // components: {
  //   Avatar,
  // },
  setup() {
    const loading = ref(true)
    const circle_data = ref("")

    const listToggle = [
    {
      label : 'Sells Commision',
      value : 'SellsCommision'
    },
    {
      label : 'Sells Comic',
      value : 'SellsComic'
    },
    {
      label : 'Sells Artbook',
      value : 'SellsArtbook'
    },
    {
      label : 'Sells Photobook',
      value : 'SellsPhotobook'
    },
    {
      label : 'Sells Novel',
      value : 'SellsNovel'
    },
    {
      label : 'Sells Game',
      value : 'SellsGame'
    },
    {
      label : 'Sells Music',
      value : 'SellsMusic'
    },
    {
      label : 'Sells Goods',
      value : 'SellsGoods'
    }]

    const listInput = [
    {
      label : 'Link Facebook',
      value : 'circle_facebook'
    },
    {
      label : 'Link Instagram',
      value : 'circle_instagram'
    },
    {
      label : 'Link twitter',
      value : 'circle_twitter'
    },
    {
      label : 'Link Medsos Lainnya',
      value : 'circle_other_socials'
    },
    {
      label : 'Link Ke Toko Circle',
      value : 'marketplace_link'
    },
    {
      label : 'Fandom Lainnya',
      value : 'other_fandom'
    },
    ]

    async function getCircleData() {
      try {
        loading.value = true
        store.user = supabase.auth.user()
        if(store.user?.id) {
          let { data, error, status } = await supabase
            .from("circle_data")
            .select('name,SellsCommision,SellsComic,SellsArtbook,SellsPhotobook,SellsNovel,SellsGame,SellsMusic,SellsGoods,circle_facebook,circle_instagram,circle_twitter,circle_other_socials,marketplace_link,other_fandom')
            .eq("user_id", store.user.id)
            .single()

          if (error && status !== 406) throw error

          if (data) {
            circle_data.value = data;
        }
        }
      } catch (error) {
        alert(`profile ${error.message}`)
      } finally {
        loading.value = false
      }
    }

    async function updateProfile() {
      try {
        loading.value = true
        const { data, error } = await supabase
          .from('circle_data')
          .update({ other_fandom: circle_data.value.other_fandom })
          .match({ user_id: store.user.id })

        if (error) throw error
      } catch (error) {
        alert(error.message)
      } finally {
        loading.value = false
      }
    }

    async function signOut() {
        loading.value = true
        let { error } = await supabase.auth.signOut()
        store.user = {};
        loading.value = false
    }

    onMounted(() => {
      getCircleData()
    })

    return {
      store,
      loading,
      listToggle,
      listInput,
      circle_data,
      getCircleData,
      updateProfile,
      signOut,
    }
  },
}
</script>
