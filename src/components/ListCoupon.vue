<template>
  <div v-if="time_all <= 0" id="list-coupon">
    <ul class="list-group">
      <li class="list-group-item active" aria-current="true">
        Mã coupon hoặc quà Quý khách đã giựt được
      </li>
      <li class="list-group-item">
        🎁 QK nhận được quà <b></b>
        <button class="btn btn-warning btn-sm">Đổi quà</button>
      </li>
      <li
        v-for="coupon in couponArray"
        :key="coupon._id"
        :coupon="coupon"
        class="list-group-item"
      >
        🎁 QK nhận được mã coupon <b>{{ coupon }}</b>
        <button
          @click="clipboardCopy(coupon)"
          class="btn btn-secondary btn-sm"
          data-toggle="tooltip1"
          data-placement="top"
          title="Copied"
        >
          Copy mã
        </button>
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
export default {
  name: "ListCoupon",
  props: {
    couponArray: { type: Array },
    time_all: { type: Number },
  },
  // @ts-ignore
  setup(props, context) {
    console.log("time", props.time_all);
    async function clipboardCopy(coupon: string) {
      console.log("click");
      await navigator.clipboard.writeText(coupon);
    }

    // @ts-ignore
    $(function () {
      // @ts-ignore
      $('[data-toggle="tooltip1"]').tooltip({
        trigger: "click",
        delay: { show: 40, hide: 60 },
      });
    });

    document.addEventListener("paste", async (e) => {
      e.preventDefault();
      const text = await navigator.clipboard.readText();
      console.log("Pasted text: ", text);
    });
    return { clipboardCopy };
  },
};
</script>

<style>
#list-coupon {
  position: fixed;
  right: 20px;
  bottom: 20px;
  z-index: 50;
}
button {
  margin-left: 8px;
}
</style>
