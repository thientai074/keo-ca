<template>
  <div id="myModal" class="modal fade" role="dialog">
    <div class="modal-dialog">
      <!-- Modal content-->
      <div class="modal-content">
        <div class="modal-body">
          <div id="msg-gift" class="text-center py-3">
            <h4 class="modal-title mb-2">
              Cám ơn Quý khách đã tham gia sự kiện
            </h4>
            <h3 class="text-danger mb-3">Trò chơi đã kết thúc</h3>
          </div>
          <div class="action text-center mb-3">
            <button class="btn btn-primary">
              <a
                class="Facebook-button"
                href="https://www.facebook.com/sharer/sharer.php?u=example.org"
                target="_blank"
              >
                Chia sẻ trên Facebook
              </a>
            </button>
          </div>
          <div class="alert alert-warning mb-0">
            <p class="m-0">
              Coupon có giá trị sử dụng đến hết ngày <b>31/08/2022</b>
            </p>
          </div>
        </div>
        <div class="modal-footer">
          <button
            @click="reloadPage()"
            type="button"
            class="btn btn-warning"
            data-dismiss="modal"
          >
            Tiếp tục
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
export default {
  name: "GiftModal",
  props: ["couponR", "couponRN"],
  // @ts-ignore
  setup(props, context) {
    function reloadPage() {
      window.location.reload();
    }
    // @ts-ignore
    $(document).ready(function () {
      // @ts-ignore
      $('[data-toggle="tooltip"]').tooltip({
        trigger: "click",
        delay: { show: 40, hide: 60 },
      });
    });

    async function clipboardCopy() {
      await navigator.clipboard.writeText(props.couponR);
    }

    document.addEventListener("paste", async (e) => {
      e.preventDefault();
      const text = await navigator.clipboard.readText();
      console.log("Pasted text: ", text);
    });
    return { clipboardCopy, reloadPage };
  },
};
</script>

<style>
.Facebook-button {
  text-decoration: none;
  color: white;
}
.Facebook-button:hover {
  text-decoration: none;
  color: white;
}
</style>
