<template>
  <form class="form-widget" @submit.prevent="updateProfile">
    <!-- <Avatar v-model:path="circle_cut" @upload="updateProfile" /> -->
    <div>Hi PJ Circle {{ circle_data.name }}</div>
    <div>
      Toggle which product types that your circle will showcase on the catalog
    </div>
    <div class="row">
      <div class="col-6-sm" v-for="toggle in listToggle" :key="toggle.value">
        <label>{{ toggle.label }}</label>
        <input
          class="tgl tgl-light"
          :id="toggle.value"
          type="checkbox"
          v-model="circle_data[toggle.value]"
        />
        <label class="tgl-btn" :for="toggle.value"> </label>
      </div>
    </div>

    <div>
      Paste below complete link for your social media links, Example :
      https://www.facebook.com/Comifuro/ , https://twitter.com/comifuro
    </div>

    <div class="row" v-for="input in listInput" :key="input.value">
      <div class="col-12-sm">
        <label :for="input.value">{{ input.label }}</label>
        <input
          :id="input.value"
          type="text"
          v-model="circle_data[input.value]"
        />
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
    <h2>
      Choose Sampleworks to upload, you can upload up to 10 images for the
      catalog. You can drag and drop images to sort the image to your need
    </h2>
    <div
      ref="parent"
      :style="{
        display: 'flex',
        'flex-direction': 'column',
        flex: 1,
        gap: '16px',
      }"
    >
      <h2 v-if="uploadLoading">Image are being uploaded, please wait</h2>
      <div
        v-for="sampleworks_image in sampleworks_images_list"
        :key="sampleworks_image"
        :style="{
          display: 'flex',
          flex: 1,
          'justify-content': 'center',
          border: 'solid',
          borderColor: 'white',
          cursor: 'grab',
        }"
      >
        <img
          :src="sampleworks_image"
          alt="Avatar"
          class="avatar image"
          :style="{ width: '100%', height: '200px', 'object-fit': 'contain' }"
        />
        <button
          class="remove-image-button"
          @click="($event) => removeImage(sampleworks_image, $event)"
        >
          Remove This Image
        </button>
      </div>
    </div>

    <div v-if="sampleworks_images_list.length < 10" class="add-image-row">
      <h2>
        Choose Samplework files: You can upload additional
        {{ 10 - sampleworks_images_list.length }} images. Selected files will be
        uploaded and you can sort the order. The changes will be reflected
        immediately in the catalog.
      </h2>
      <input
        type="file"
        multiple
        @change="chooseSampleworks"
        name="file"
        accept="image/*"
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
import { supabase } from "../supabase";
import { store } from "../store";
import { onMounted, ref, watch } from "vue";
import { compileScript } from "@vue/compiler-sfc";
// import Avatar from "./Avatar.vue"
import { useDragAndDrop } from "@formkit/drag-and-drop/vue";
import imageReducer from "image-blob-reduce";

export default {
  // components: {
  //   Avatar,
  // },
  setup() {
    const loading = ref(true);
    const uploadLoading = ref(false);
    const circle_data = ref("");

    const listToggle = [
      {
        label: "Sells Commision",
        value: "SellsCommision",
      },
      {
        label: "Sells Comic",
        value: "SellsComic",
      },
      {
        label: "Sells Artbook",
        value: "SellsArtbook",
      },
      {
        label: "Sells Photobook General",
        value: "SellsPhotobookGeneral",
      },
      {
        label: "Sells Photobook Cosplay",
        value: "SellsPhotobookCosplay",
      },
      {
        label: "Sells Novel",
        value: "SellsNovel",
      },
      {
        label: "Sells Game",
        value: "SellsGame",
      },
      {
        label: "Sells Music",
        value: "SellsMusic",
      },
      {
        label: "Sells Goods",
        value: "SellsGoods",
      },
      {
        label: "Sells Handmade Crafts",
        value: "SellsHandmadeCrafts",
      },
      {
        label: "Sells Magazine",
        value: "SellsMagazine",
      },
    ];

    const listInput = [
      {
        label: "Link Facebook",
        value: "circle_facebook",
      },
      {
        label: "Link Instagram",
        value: "circle_instagram",
      },
      {
        label: "Link twitter",
        value: "circle_twitter",
      },
      {
        label: "Link Medsos Lainnya",
        value: "circle_other_socials",
      },
      {
        label: "Fandom Lainnya",
        value: "other_fandom",
      },
    ];

    const circle_samples = ref("");

    function blobToFile(theBlob, fileName) {
      return new File([theBlob], fileName, {
        lastModified: new Date().getTime(),
        type: theBlob.type,
      });
    }

    async function getCircleData() {
      try {
        loading.value = true;
        store.user = supabase.auth.user();
        if (store.user?.id) {
          let { data, error, status } = await supabase
            .from("circle_data")
            .select(
              "name,circle_code,SellsCommision,SellsComic,SellsArtbook,SellsPhotobookGeneral,SellsNovel,SellsGame,SellsMusic,SellsGoods,circle_facebook,circle_instagram,circle_twitter,circle_other_socials,marketplace_link,other_fandom,sampleworks_images,SellsHandmadeCrafts,SellsMagazine,SellsPhotobookCosplay"
            )
            .eq("user_id", store.user.id)
            .single();

          if (error && status !== 406) throw error;

          if (data) {
            circle_data.value = data;
            sampleworks_images_list.value = data.sampleworks_images
              ? data.sampleworks_images
              : [];
          }
        }
      } catch (error) {
        alert(`profile ${error.message}`);
      } finally {
        loading.value = false;
      }
    }

    async function removeImage(sampleworks_image, $event) {
      $event.preventDefault();
      const { data, error: error_upload } = await supabase.storage
        .from("circle-sampleworks-18")
        .remove([sampleworks_image.slice(88)], {
          cacheControl: "3600",
          upsert: true,
        });

      const filteredSampleworks = sampleworks_images_list.value.filter(
        (image) => image !== sampleworks_image
      );

      sampleworks_images_list.value = filteredSampleworks;
    }

    async function chooseSampleworks(event) {
      uploadLoading.value = true;
      const imageLimit = 10 - sampleworks_images_list.value.length;

      if (event.target.files.length > imageLimit) {
        event.target.value = null;
        alert(
          `You can only select up to ${imageLimit} images, because you already uploaded ${sampleworks_images_list.value.length} images`
        );
        uploadLoading.value = false;
        return;
      }
      try {
        loading.value = true;
        for (let index = 0; index < event.target.files.length; index++) {
          const sampleFile = event.target.files[index];
          console.log(sampleFile);
          const filename = sampleFile.name.split(".").pop();

          let reduce = new imageReducer();
          let resizeFile = null;

          await reduce.toBlob(sampleFile, { max: 3000 }).then((blob) => {
            resizeFile = blobToFile(blob, filename);
          });

          const file_folder = self.crypto.randomUUID();

          const { data, error: error_upload } = await supabase.storage
            .from("circle-sampleworks-18")
            .upload(
              `${store.user.id}/${file_folder}/${circle_data.value.circle_code}.${filename}`,
              resizeFile,
              {
                cacheControl: "3600",
                upsert: true,
              }
            );

          sampleworks_images_list.value.push(
            `https://kumxjefxtrrpzalmwvvr.supabase.in/storage/v1/object/public/${data.Key}`
          );

          if (error_upload) throw error_upload;
          const { data: data_update, error: error_update } = await supabase
            .from("circle_data")
            .update({
              sampleworks_images: sampleworks_images_list.value,
            })
            .match({ user_id: store.user.id });
          if (error_update) throw error_update;
        }

        // console.log(data);

        // if (data) alert("Sampleworks Circle Berhasil Diperbarui ");
      } catch (error) {
        alert(error.message);
      } finally {
        event.target.value = null;
        loading.value = false;
        uploadLoading.value = false;
      }
    }

    async function updateProfile() {
      try {
        loading.value = true;
        console.log(circle_data.value);
        const { sampleworks_images, ...circleData } = circle_data.value;
        const { data, error } = await supabase
          .from("circle_data")
          .update({ ...circleData })
          .match({ user_id: store.user.id });

        if (error) throw error;
        if (data) alert("Data Circle Berhasil Diperbarui");
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    }

    async function signOut() {
      loading.value = true;
      let { error } = await supabase.auth.signOut();
      store.user = {};
      loading.value = false;
    }

    onMounted(() => {
      getCircleData();
    });

    const [parent, sampleworks_images_list] = useDragAndDrop([]);

    watch(
      sampleworks_images_list,
      async (sampleworks_images_list, prevSampleworks_images_list) => {
        if (prevSampleworks_images_list.length !== 0) {
          const { data: data_update, error: error_update } = await supabase
            .from("circle_data")
            .update({
              sampleworks_images: sampleworks_images_list,
            })
            .match({ user_id: store.user.id });

          circle_data.value.sampleworks_images = data_update;
        }
      }
    );

    return {
      store,
      loading,
      uploadLoading,
      listToggle,
      listInput,
      circle_data,
      getCircleData,
      updateProfile,
      chooseSampleworks,
      parent,
      sampleworks_images_list,
      signOut,
      removeImage,
    };
  },
};
</script>
