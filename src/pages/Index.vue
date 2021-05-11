<template>
  <q-page class="flex flex-center" v-on:keypress="keypressHandler">
    <section>
      <q-input
        style="font-size: 2rem"
        size="large"
        v-if="showing"
        id="input"
        rounded
        outlined
        v-model="newInput"
        autofocus
        @keyup="setTextInput"
      >
        <template v-slot:append>
          <q-avatar>
            <img src="~assets/backslash-logo.png" />
          </q-avatar>
        </template>
      </q-input>
      <section class="start">
        <p>{{ !showing ? "Press / to start" : "" }}</p>
        <p>{{ showing ? "\`Shift\` + \`Space\` to toggle" : "" }}</p>
        <p>
          {{
            showing && containsKeyword !== "help"
              ? "Type \`help\` for assistance"
              : ""
          }}
        </p>
        <p>
          {{
            showing && containsKeyword !== "list"
              ? "Type \`list\` for commands"
              : ""
          }}
        </p>
        <p>{{ showing ? "Press \`esc\` to close" : "" }}</p>
      </section>
      <section v-if="containsKeyword === 'help'">
        <p><strong>Modifier Key:</strong>{{ " " }} <code>shift</code></p>
        <strong>Special Keys:</strong>{{ " " }}
        <q-item
          clickable
          dense
          v-ripple
          v-for="item in specialCharList"
          :key="item.key"
        >
          <q-item-section>
            <q-item><q-icon :name="item.icon" size="md"/></q-item>
            <q-item-label>{{ item.key }} - {{ item.action }}</q-item-label>
            <q-item-label caption>{{ item.description }}</q-item-label>
          </q-item-section>
        </q-item>
      </section>

      <section v-else-if="containsKeyword === 'list'">
        <strong>Keywords:</strong>{{ " " }}
        <q-item clickable dense v-ripple v-for="(item, i) in keywords" :key="i">
          <q-item-section>
            <q-item-label>{{ item }}</q-item-label>
          </q-item-section>
        </q-item>
      </section>
    </section>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      newInput: "",
      textInput: "",
      showing: false,
      specialCharList: [
        {
          key: "/",
          action: "general",
          description: "access all commands",
          icon: "mdi-slash-forward-box"
        },
        {
          key: ">",
          action: "options",
          description: "create actions or personalize your action page",
          icon: "mdi-code-greater-than"
        },
        {
          key: "!",
          action: "action",
          description: "dispatch an action",
          icon: "mdi-alert-box"
        },
        {
          key: "?",
          action: "information",
          description: "get general information or lookup command information",
          icon: "mdi-help-box"
        }
      ],
      keywords: [
        "note",
        "list",
        "help",
        "www.",
        ".com",
        "search",
        "mail",
        "go",
        "do",
        "*",
        "/",
        "+",
        "-"
      ]
    };
  },
  created() {
    window.addEventListener("keydown", e => {
      this.keypressHandler(e, "down");
    });
    window.addEventListener("keyup", e => {
      this.keypressHandler(e, "up");
    });
  },
  watch: {
    containsKeyword(newValue, oldValue) {
      console.log(oldValue);
      if (newValue?.length > 1) {
        this.$store.dispatch("setFooter", newValue);
      }
    }
  },
  methods: {
    setTextInput() {
      this.textInput = this.filterInput(this.newInput.trim()).trim();
    },
    filterInput(value) {
      value = value.trim();
      let firstChar = value[0];
      if (this.specialChars.includes(firstChar)) {
        return value.replace(firstChar, "");
      } else {
        return value;
      }
    },
    focusOnInput(e) {
      if (this.newInput.length === 2) {
        this.clearInput(false);
        this.newInput = e.key;
        this.setTextInput(e.key);
      }
      document.querySelector("input").focus();
    },
    keypressHandler(e, dir) {
      if (this.showing === true) {
        if (e.key === "Escape") {
          this.clearInput(true);
        } else if (this.specialChars.includes(e.key)) {
          this.focusOnInput(e);
        } else if (dir === "down" && e.shiftKey && e.code === "Space") {
          this.clearInput(true);
        } else return;
      } else {
        if (this.specialChars.includes(e.key)) {
          this.showing = true;
        } else if (dir === "down" && e.shiftKey && e.code === "Space") {
          this.showing = true;
          this.clearInput(false);
        }
      }
    },
    clearInput(hide) {
      this.$store.dispatch("setFooter", "backslash");
      if (!hide) {
        this.newInput = "";
        this.setTextInput("");
      } else {
        this.newInput = "";
        this.setTextInput("");
        this.showing = false;
      }
    },
    toggleInputView() {
      this.showing = !this.showing;
    },
    searchForWord(searchFor, within) {
      return searchFor.includes(within);
    },
    matchWord(matchThis, withThis) {
      return matchThis === withThis;
    }
  },
  computed: {
    specialChars() {
      return this.specialCharList.map(specialChar => specialChar.key);
    },
    specialCharInfo() {
      return this.specialCharList.map(specialChar => {
        return `${specialChar.key} - ${specialChar.action}`;
      });
    },
    containsKeyword() {
      if (this.textInput === "") {
        return false;
      } else {
        return this.keywords.find(keyword => {
          return this.textInput?.toLowerCase().includes(keyword);
        });
      }
    }
  }
};
</script>

<style>
li {
  list-style: none;
}
</style>