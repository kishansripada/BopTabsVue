<template>
  <div
    class="container d-flex p-0 shadow-lg rounded-3 mt-4 position-relative"
    style="height: 200px"
  >
    <h1
      style="
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        padding: 10px;
        text-align: center;
      "
    >
      {{
        `There are no ${tabsOrChords} for this song 😪 ${
          tabsOrChords == "tabs" ? "Upload" : "Write"
        } your own!`
      }}
    </h1>
    <img
      style="
        position: absolute;
        height: 150px;
        right: 5%;
        transform: rotate(20deg);
      "
      src="https://raw.githubusercontent.com/kishansripada/BopTabs/master/src/assets/arrow.png"
    />
  </div>
</template>
<script>
import { mapState } from "vuex";
export default {
  name: "NoTabsNoChords",
  computed: {
    ...mapState(["tabsOrChords"]),
  },
};
</script>