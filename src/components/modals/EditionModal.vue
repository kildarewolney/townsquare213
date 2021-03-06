<template>
  <Modal
    class="editions"
    v-show="modals.edition"
    @close="toggleModal('edition')"
  >
    <div v-if="!isCustom">
      <h3>Select an edition:</h3>
      <ul class="editions">
        <li
          v-for="edition in editions"
          class="edition"
          v-bind:class="['edition-' + edition.id]"
          v-bind:key="edition.id"
          @click="setEdition(edition.id)"
        >
          {{ edition.name }}
        </li>
        <li class="edition edition-custom" @click="isCustom = true">
          Custom Script / Characters
        </li>
      </ul>
    </div>
    <div class="custom" v-else>
      <h3>Load custom script / characters</h3>
      Para jogar com um custom script, você precisa selecionar os personagens que você quer
      para jogar com a ferramenta oficial
      <a href="https://bloodontheclocktower.com/script-tool/" target="_blank"
        >Script Tool</a
      >
      e fazer o upload do "custom-list.json" gerado diretamente aqui ou
      forneça uma URL para esse arquivo JSON hospedado..<br />
      <br />
      Para jogar com personagens personalizados, por favor leia
      <a
        href="https://github.com/bra1n/townsquare#custom-characters"
        target="_blank"
        >the documentation</a
      >
      sobre como escrever um arquivo de definição de caracteres personalizados.
      <h3>Alguns Custom Scripts Populares:</h3>
      <ul class="scripts">
        <li
          v-for="(script, index) in scripts"
          v-bind:key="index"
          @click="handleURL(script[1])"
        >
          {{ script[0] }}
        </li>
      </ul>
      <input
        type="file"
        ref="upload"
        accept="application/json"
        @change="handleUpload"
      />
      <div class="button-group">
        <div class="button" @click="openUpload">
          <font-awesome-icon icon="file-upload" /> Upload JSON
        </div>
        <div class="button" @click="promptURL">
          <font-awesome-icon icon="link" /> Enter URL
        </div>
        <div class="button" @click="isCustom = false">
          <font-awesome-icon icon="undo" /> Voltar
        </div>
      </div>
    </div>
  </Modal>
</template>

<script>
import editionJSON from "../../editions";
import { mapMutations, mapState } from "vuex";
import Modal from "./Modal";

export default {
  components: {
    Modal
  },
  data: function() {
    return {
      editions: editionJSON,
      isCustom: false,
      scripts: [
        [
          "Deadly Penance Day",
          "https://gist.githubusercontent.com/bra1n/0337cc44c6fd2c44f7589256ed5486d2/raw/4a7a1545004620146f47583cde4b05f77dd9b6d2/penanceday.json"
        ],
        [
          "Catfishing 9.0",
          "https://gist.githubusercontent.com/bra1n/8a5ec41a7bbf945f6b7dfc1cef72b569/raw/998767f82badc48cbb9c284765ad36330f7e28f6/catfishing.json"
        ],
        [
          "On Thin Ice (Teensyville)",
          "https://gist.githubusercontent.com/bra1n/8dacd9f2abc6f428331ea1213ab153f5/raw/0cacbcaf8ed9bddae0cca25a9ada97e9958d868b/on-thin-ice.json"
        ],
        [
          "Race To The Bottom (Teensyville)",
          "https://gist.githubusercontent.com/bra1n/63e1354cb3dc9d4032bcd0623dc48888/raw/5acb0eedcc0a67a64a99c7e0e6271de0b7b2e1b2/race-to-the-bottom.json"
        ],
        [
          "Frankenstein's Mayor by Ted (Teensyville)",
          "https://gist.githubusercontent.com/bra1n/32c52b422cc01b934a4291eeb81dbcee/raw/5bf770693bbf7aff5e86601c82ca4af3222f4ba6/Frankensteins_Mayor_by_Ted.json"
        ],
        [
          "Vigormortis High School (Teensyville)",
          "https://gist.githubusercontent.com/bra1n/1f65bd4a999524719d5dabe98c3c2d27/raw/22bbec6bf56a51a7459e5ae41ed47e41971c5445/VigormortisHighSchool.json"
        ],
        [
          "Red The Room",
          "https://gist.githubusercontent.com/kildarewolney/3397ae5dd4f9747cf4c88823522e0b18/raw/b8aa4511b0756446738d35431bd824c79f240a62/gistfile1.json"
        ]
      ]
    };
  },
  computed: mapState(["modals"]),
  methods: {
    openUpload() {
      this.$refs.upload.click();
    },
    handleUpload() {
      const file = this.$refs.upload.files[0];
      if (file && file.size) {
        const reader = new FileReader();
        reader.addEventListener("load", () => {
          try {
            const roles = JSON.parse(reader.result);
            this.parseRoles(roles);
          } catch (e) {
            alert("Error reading custom script: " + e.message);
          }
          this.$refs.upload.value = "";
        });
        reader.readAsText(file);
      }
    },
    promptURL() {
      const url = prompt("Enter URL to a custom-script.json file");
      if (url) {
        this.handleURL(url);
      }
    },
    async handleURL(url) {
      const res = await fetch(url);
      if (res && res.json) {
        try {
          const script = await res.json();
          this.parseRoles(script);
        } catch (e) {
          alert("Error loading custom script: " + e.message);
        }
      }
    },
    parseRoles(roles) {
      if (!roles || !roles.length) return;
      const customRoles = roles.map(role => {
        role.id = role.id.toLocaleLowerCase().replace(/[^a-z0-9]/g, "");
        return role;
      });
      this.$store.commit("setCustomRoles", customRoles);
      this.$store.commit("setEdition", "custom");
      // check for fabled and set those too, if present
      if (customRoles.some(({ id }) => this.$store.state.fabled.has(id))) {
        const fabled = [];
        customRoles.forEach(({ id }) => {
          if (this.$store.state.fabled.has(id)) {
            fabled.push(this.$store.state.fabled.get(id));
          }
        });
        this.$store.commit("setFabled", { fabled });
      }
      this.isCustom = false;
    },
    ...mapMutations(["toggleModal", "setEdition"])
  }
};
</script>

<style scoped lang="scss">
@import "../../vars";

// Editions
@each $img, $skipIcons in $editions {
  .edition-#{$img} {
    background-image: url("../../assets/editions/#{$img}.png");
  }
  @if $skipIcons != true {
    .edition-#{$img}.townsfolk {
      background-image: url("../../assets/editions/#{$img}-townsfolk.png");
    }
    .edition-#{$img}.outsider {
      background-image: url("../../assets/editions/#{$img}-outsider.png");
    }
    .edition-#{$img}.minion {
      background-image: url("../../assets/editions/#{$img}-minion.png");
    }
    .edition-#{$img}.demon {
      background-image: url("../../assets/editions/#{$img}-demon.png");
    }
  }
}

ul.editions .edition {
  font-family: PiratesBay, sans-serif;
  letter-spacing: 1px;
  text-align: center;
  padding-top: 15%;
  background-position: center center;
  background-size: 100% auto;
  background-repeat: no-repeat;
  width: 30%;
  margin: 5px;
  font-size: 120%;
  text-shadow: -1px -1px 0 #000, 1px -1px 0 #000, -1px 1px 0 #000,
    1px 1px 0 #000, 0 0 5px rgba(0, 0, 0, 0.75);
  cursor: pointer;
  &:hover {
    color: red;
  }
}

.custom {
  text-align: center;
  input[type="file"] {
    display: none;
  }
  .scripts {
    list-style-type: disc;
    font-size: 120%;
    cursor: pointer;
    display: block;
    width: 50%;
    text-align: left;
    margin: 10px auto;
    li:hover {
      color: red;
    }
  }
}
</style>
