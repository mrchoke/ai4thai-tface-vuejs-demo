<template>
  <v-container>
    <v-card>
      <v-toolbar>
        <v-toolbar-title>
          T-Face API Vue.js Demo
        </v-toolbar-title>
      </v-toolbar>
      <v-row>
        <v-col cols="4">
          <v-img
            :src="output ? output : img ? img : noprofile"
            cover
            @click="$refs.addFile.click()"
            width="70%"
            class="mx-auto"
          ></v-img>
        </v-col>
        <v-col cols="8">
          <input
            type="file"
            accept="image/jpeg,image/png"
            @change="preview($event)"
            ref="addFile"
            id="addFile"
          />

          <v-row>
            <v-col>
              <v-btn
                outlined
                color="primary"
                large
                @click="$refs.addFile.click()"
                block
              >
                <span class="title">
                  {{ img ? "เปลี่ยน" : "เลือก" }}รูปภาพ
                </span>
                <v-icon>
                  mdi-image-plus
                </v-icon>
              </v-btn>
            </v-col>
          </v-row>
          <v-row v-if="hasError">
            <v-col>
              <v-alert
                :color="status_type"
                dark
                :icon="status_icon"
                border="left"
                prominent
              >
                {{ status_message }}
              </v-alert>
            </v-col>
          </v-row>
          <v-row v-if="search && search.boxes">
            <v-col cols="12">
              <v-toolbar> ค้นพบ {{ boxes.length }} ใบหน้า </v-toolbar>
            </v-col>
            <v-col>
              <v-row v-for="(item, i) in search.result" :key="i">
                <v-col class="d-flex child-flex">
                  <v-card
                    tile
                    max-width="300"
                    :style="
                      `border-width:1px;border-color: ${color(
                        i
                      )};border-style: solid;border-radius: 5px;`
                    "
                  >
                    <v-img
                      :src="cropFace(boxes[i])"
                      aspect-ratio="1"
                      class="white--text align-end"
                    >
                      <template v-slot:placeholder>
                        <v-row
                          class="fill-height ma-0"
                          align="center"
                          justify="center"
                        >
                          <v-progress-circular
                            indeterminate
                            color="grey lighten-5"
                          ></v-progress-circular>
                        </v-row>
                      </template>
                    </v-img>
                    <v-card-subtitle class="text-center text-h6">
                      หน้าที่ {{ i + 1 }}
                    </v-card-subtitle>
                  </v-card>
                </v-col>
                <v-col class="d-flex child-flex" v-if="item && item.length">
                  <v-card
                    tile
                    max-width="300"
                    :style="
                      `border-width:1px;border-color: ${color(
                        i
                      )};border-style: solid;border-radius: 5px;`
                    "
                  >
                    <v-img
                      :src="`${img_url}${item[0].image_path}`"
                      aspect-ratio="1"
                      class="white--text align-end"
                    >
                      <template v-slot:placeholder>
                        <v-row
                          class="fill-height ma-0"
                          align="center"
                          justify="center"
                        >
                          <v-progress-circular
                            indeterminate
                            color="grey lighten-5"
                          ></v-progress-circular>
                        </v-row>
                      </template>
                    </v-img>
                    <v-card-subtitle class="text-center text-h6">
                      {{ parseFloat(item[0].score * 100).toFixed(2) }}
                    </v-card-subtitle>
                  </v-card>
                </v-col>
              </v-row>
            </v-col>
          </v-row>
          <v-row v-if="searching">
            <v-col class="text-center">
              <v-icon size="100" color="#043c3b" class="mdi-spin"
                >mdi-loading</v-icon
              >
            </v-col>
          </v-row>
        </v-col>
      </v-row>
    </v-card>
  </v-container>
</template>

<script>
import loadImage from "blueimp-load-image";
import NoProfile from "./NoProfile";

export default {
  name: "TFace",
  data: () => {
    return {
      searching: false,
      img: null,
      output: null,
      noprofile: NoProfile,
      hiddencanvas: null,
      search: {},
      boxes: [],
      image: {},
      hasError: false,
      status_type: "info",
      status_message: "",
      status_icon: "info",
      project_id: "",
      api_key: "",
      img_url:
        "https://www.aiforthai.in.th/aiplatform/views/pages/facetraining/upload/"
    };
  },
  computed: {
    url() {
      return `https://api.aiforthai.in.th/t-face/base64/${this.project_id}`;
    }
  },
  created() {
    this.hiddencanvas = document.createElement("canvas");
  },
  watch: {
    img: function(v) {
      if (!v || v === NoProfile) {
        return;
      }
      this.searchApi();
      let c = 0;
      let id = setInterval(() => {
        c++;
        try {
          let canvas = this.hiddencanvas;
          let ctx = canvas.getContext("2d");
          let image = new Image();
          image.onload = function() {
            canvas.height = image.height;
            canvas.width = image.width;
            ctx.drawImage(image, 0, 0);
          };
          image.src = v;
          this.image = image;

          clearInterval(id);
        } catch (e) {
          if (c > 100) clearInterval(id);
        }
      }, 25);
    },
    boxes(v) {
      if (v && v.length) this.drawResults(v);
    }
  },

  methods: {
    color(n) {
      const colors = [
        "#0059ff",
        "#ff0044",
        "#10e372",
        "#ff9c08",
        "#a408ff",
        "#00ddfa",
        "#ffdd00",
        "#b5ed0c",
        "#8a5a24",
        "#c918ac"
      ];

      let ns = n.toString();
      let i = ns[ns.length - 1];

      return colors[i];
    },

    async searchApi() {
      this.search = {};
      this.output = null;
      this.boxes = [];
      this.searching = true;
      await fetch(this.url, {
        method: "POST",
        body: JSON.stringify({ image: this.img.split(",")[1] }),
        headers: {
          "Content-Type": "application/json",
          Apikey: this.api_key
        }
      })
        .then(async res => {
          if (!res.ok) {
            this.hasError = true;
            this.status_type = "error";
            this.status_icon = "mdi-alert-circle-outline";
            if (res.status >= 400 && res.status <= 499) {
              this.status_message = `${res.status}: ส่งข้อมูลให้ API ไม่ถูกต้อง`;
            }

            if (res.status >= 500 && res.status <= 599) {
              this.status_message = `${res.status}: API ไม่สามารถทำงานได้`;
            }
          } else {
            const result = await res.json();
            if (result && result.boxes) {
              this.search = result;
              this.boxes = result.boxes;
              this.hasError = false;
            } else {
              this.searchng = false;
              this.hasError = true;
              this.status_type = "error";
              this.status_icon = "mdi-alert-circle-outline";
              this.status_message = "ไม่สามารถตรวจจับใบหน้าได้";
            }
            this.searching = false;
          }
        })
        .catch(error => {
          this.status_type = "error";
          this.status_icon = "mdi-alert-circle-outline";
          this.status_message = error;
        })
        .finally(() => {
          this.searching = false;
        });
    },
    drawResults(results) {
      let canvas = this.hiddencanvas;
      let ctx = canvas.getContext("2d");

      for (let i = 0; i < results.length; ++i) {
        let rect = results[i];
        ctx.beginPath();
        let xRatio = ctx.canvas.width / this.image.width;
        let yRatio = ctx.canvas.height / this.image.height;
        ctx.lineWidth = 3;
        ctx.strokeStyle = this.color(i);

        ctx.strokeRect(
          rect.x * xRatio,
          rect.y * yRatio,
          rect.w * xRatio,
          rect.h * yRatio
        );
      }
      this.output = canvas.toDataURL("image/jpeg");
    },
    cropFace(rect) {
      let margin = 40;
      let canvas = document.createElement("canvas");
      let ctx = canvas.getContext("2d");
      canvas.width = rect.w + margin * 2;
      canvas.height = rect.h + margin * 2;
      let image = new Image();
      image.onload = function() {};
      image.src = this.img;

      let x = rect.x - margin;
      let y = rect.y - margin;
      let h = rect.h + margin * 2;
      let w = rect.w + margin * 2;

      ctx.drawImage(image, x, y, w, h, 0, 0, w, h);
      let data = canvas.toDataURL();

      return data;
    },

    preview(e) {
      this.hasError = false;
      this.img = null;
      this.output = null;
      let file = (this.files = e.target.files[0] || e.target.files);

      loadImage.parseMetaData(file, data => {
        const options = {
          canvas: true,
          maxHeight: 640,
          maxWidth: 640
        };
        if (data.exif) {
          options.orientation = data.exif.get("Orientation");
        }
        this.displayImage(file, options);
      });
    },
    displayImage(file, options) {
      loadImage(
        file,
        async canvas => {
          this.img = canvas.toDataURL("image/jpeg");
        },
        options
      );
    }
  }
};
</script>

<style scoped>
#addFile {
  display: none;
}
</style>
