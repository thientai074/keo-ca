<template>
  <div>
    <canvas id="canvas1" width="800" height="500"></canvas>
    <GiftModal :couponR="couponR" :couponRN="couponRN" />
    <ListCoupon :couponArray="couponArray" :time_all="time_all" />
  </div>
</template>

<script lang="ts">
import { onMounted, Ref, ref } from "vue";
import GiftModal from "./GifModal.vue";
import ListCoupon from "./ListCoupon.vue";
export default {
  name: "Home",
  components: { GiftModal, ListCoupon },

  setup() {
    onMounted(() => {
      draw();
    });

    const couponR: Ref<string | undefined> = ref("");
    const couponRN: Ref<number | undefined> = ref();
    const couponArray = ref([]);
    const time_all: any = ref(6);

    function draw() {
      // Canvas setup
      const canvas: any = document.getElementById("canvas1");
      const ctx = canvas.getContext("2d");

      canvas.width = windowWH().w - 100;
      canvas.height = windowWH().h - 100;

      ctx.font = "50px Georgia";

      var canvasPosition = canvas.getBoundingClientRect();

      interface FishType {
        x: number;
        y: number;
        size: number;
        dx: number;
        dy: number;
        frame: number;
        frameX: number;
        frameY: number;
        spriteWidth: number;
        spriteHeight: number;
        is_catching: boolean;
        owner: any;
        r: number;
        speed: number;
        draw: () => void;
      }

      interface PlayerType {
        x: number;
        y: number;
        width: number;
        height: number;
        color: string;
        hook: any;
        draw: () => void;
      }

      let gameFrame = 0;
      var is_pulling = false;
      var game_end = false;
      var game_pause = false;
      var players: PlayerType[] = [];
      var fish_arr: FishType[] = [];
      var level: number = 1;
      var mode: number = 0;
      var catching_object: any = 0;
      var time_limit: any;
      var angleCurrent: number;

      // Function
      async function createLevel(level: number) {
        // @ts-ignore
        const mouse_move = function async(event) {
          if (!is_pulling) {
            event.preventDefault();
            // @ts-ignore
            const startX = parseInt(event.clientX - canvasPosition.left);
            // @ts-ignore
            const startY = parseInt(event.clientY - canvasPosition.top);

            const disX = startX - canvas.width / 2 - 25;
            const disY = startY - canvas.height + 54 - 50;
            // const disY = startY - 50;
            angleCurrent = (Math.atan2(disY, disX) * 180) / Math.PI;

            // Huong di cua hook
            for (var i = 0; i < players.length; i++) {
              players[i].hook.status = 0;

              if (players[i].hook.object) {
                players[i].hook.object.x = -100;
                players[i].hook.object.y = -100;
                players[i].hook.object.moveSpeed = 0;
                players[i].hook.object = 0;
              }

              if (angleCurrent) {
                players[i].hook.angle = angleCurrent;
              }

              players[i].hook.x = players[i].x + players[i].width / 2;
              players[i].hook.y = canvas.height - 4;
              players[i].hook.moveSpeed = 20;
            }
          }
        };

        canvas.onmousemove = mouse_move;

        if (time_all.value > 1) {
          setInterval(timeUpdate, 100);
        }

        switch (level) {
          case 1:
            {
              for (let i = 0; i < 6; i++) {
                fish_arr.push(
                  // @ts-ignore
                  new fish(
                    getRandomArbitrary(50, canvas.width - 50),
                    getRandomArbitrary(65, canvas.height - 65),
                    1
                  )
                );
              }
            }
            break;
        }

        canvas.onclick = function () {
          if (!game_end && !game_pause) {
            is_pulling = true;

            if (players[0].hook.status == 0) {
              // is_pulling = false
              players[0].hook.status = 1;
              if (time_all.value > 1) {
                timeUpdate();
              }
            }
          }
        };

        time_limit = setInterval(function () {
          // time_all.value--;
        }, 1000);
      }

      function clearCanvas() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
      }

      function draw_time_limit() {
        ctx.fillStyle = "gray";
        ctx.font = "bold 20px ppFont";
        ctx.fillText("Time:", canvas.width - 200, 40);
        ctx.fillText(time_all.value.toString(), canvas.width - 120, 40);
      }

      function draw_all() {
        clearCanvas();
        for (var i = 0; i < fish_arr.length; i++)
          if (fish_arr[i].owner == 0) fish_arr[i].draw();

        for (var i = 0; i < players.length; i++) {
          players[i].draw();

          if (players[i].hook.object) {
            players[i].hook.object.draw();
          }
          players[i].hook.draw();
        }
        draw_time_limit();
      }

      function check_all() {
        if (time_all.value == 0) {
          OpenBootstrapPopup();
        }

        for (var i = 0; i < players.length; i++) {
          players[i].hook.check();
        }
        check_hook_catch_object();
      }

      // Function bat trung Ca
      function check_hook_catch_object() {
        for (var i = 0; i < fish_arr.length; i++) {
          for (var j = 0; j < players.length; j++) {
            if (
              Math.sqrt(
                (fish_arr[i].x - players[j].hook.x) *
                  (fish_arr[i].x - players[j].hook.x) +
                  (fish_arr[i].y - players[j].hook.y) *
                    (fish_arr[i].y - players[j].hook.y)
              ) < fish_arr[i].r &&
              players[j].hook.status == 1 &&
              players[j].hook.moveSpeed > 0 &&
              fish_arr[i].owner == 0
            ) {
              fish_arr[i].is_catching = true;
              fish_arr[i].x =
                players[j].hook.x +
                fish_arr[i].r *
                  Math.cos((players[j].hook.angle * Math.PI) / 180);
              fish_arr[i].y =
                players[j].hook.y +
                fish_arr[i].r *
                  Math.sin((players[j].hook.angle * Math.PI) / 180);
              players[j].hook.status = 2;
              players[j].hook.object = fish_arr[i];
              fish_arr[i].owner = players[j].hook;
              players[j].hook.moveSpeed =
                -players[j].hook.moveSpeed * fish_arr[i].speed;
              couponR.value = (Math.random() + 1).toString(36).substring(7);
              couponRN.value = getRandomArbitrary(10, 50);
              // @ts-ignore
              couponArray.value.push(couponR.value);
            }
          }
        }
      }

      function timeUpdate() {
        draw_all();
        check_all();
      }

      // Player
      function player(this: any, num: number) {
        this.x = canvas.width / 2;
        this.y = canvas.height - 54;
        // this.y = 0;
        this.width = 50;
        this.height = 50;
        this.color = "blue";
        // @ts-ignore
        this.hook = new hook(this.x + this.width / 2);
        this.hook.owner = this;
        // this.hook.init();

        this.init();
      }

      player.prototype = {
        init: function () {
          this.draw();
        },
        draw: function () {
          var playerImg = new Image();
          playerImg.src = "img/player.jpeg";
          ctx.drawImage(playerImg, this.x, this.y, this.width, this.height);
        },
      };

      // Fish
      function fish(this: any, x: number, y: number, size: number) {
        this.x = x;
        this.y = y;
        this.size = size;
        this.dx = getRandomArbitrary(30, 50);
        this.dy = getRandomArbitrary(30, 50);
        this.frame = 0;
        this.frameX = 0;
        this.frameY = 0;
        this.spriteWidth = 418;
        this.spriteHeight = 397;
        this.is_catching = false;

        switch (size) {
          case 1:
            {
              //this.r = canvas.width / 40;
              this.r = 40;
              this.value = 50;
            }
            break;
        }

        this.speed = 40 / this.value;
        this.owner = 0;
        this.moveSpeed = 0;
        this.init();
      }
      fish.prototype = {
        init: function () {
          // this.draw();
        },
        draw: function () {
          if (gameFrame % 5 == 0) {
            this.frame++;
            if (this.frame >= 12) this.frame = 0;
            if (this.frame == 3 || this.frame == 7 || this.frame == 11) {
              this.frameX = 0;
            } else {
              this.frameX++;
            }
            if (this.frame < 3) this.frameY = 0;
            else if (this.frame < 7) this.frameY = 1;
            else if (this.frame < 11) this.frame = 2;
            else this.frameY = 0;
          }

          if (this.x - 58 < 0 || this.x + 58 > canvas.width) {
            this.dx = -this.dx;
          }

          if (this.y - 65 < 0 || this.y + 65 > canvas.height) {
            this.dy = -this.dy;
          }

          if (this.is_catching == false) {
            this.x += this.dx;
            this.y += this.dy;
          } else if (this.is_catching == true) {
            if (this.owner != 0) this.moveSpeed = this.owner.moveSpeed;
            if (this.moveSpeed) {
              this.x +=
                this.moveSpeed * Math.cos((this.owner.angle * Math.PI) / 180);
              this.y +=
                this.moveSpeed * Math.sin((this.owner.angle * Math.PI) / 180);
            }
          }

          var fishImg = new Image();
          fishImg.src = "img/__red_cartoon_fish_01_swim.png";
          if (this.size == 1)
            ctx.drawImage(
              fishImg,
              this.frameX * this.spriteWidth,
              this.frameY * this.spriteHeight,
              this.spriteWidth,
              this.spriteHeight,
              this.x - 60,
              this.y - 70,
              this.spriteWidth / 3,
              this.spriteWidth / 3
            );
        },
      };

      // Hook
      function hook(this: any, x: number) {
        this.x = x;
        this.y = canvas.height - 54 + 50;
        this.angle = 0;
        this.object = 0;
        this.status = 0;
        this.length = 0;
        this.moveSpeed = 20;
        this.r = 10;
        this.maxLength = 500;
        this.color = "black";
        this.owner = 0;
        // this.init();
      }

      hook.prototype = {
        init: function () {
          this.draw();
        },
        draw: function () {
          if (this.status == 0) {
            is_pulling = false;
            ctx.beginPath();
            ctx.save();
            ctx.translate(
              this.owner.x + this.owner.width / 2,
              this.owner.y + this.owner.height
            );

            ctx.lineWidth = 1;
            ctx.lineCap = "round";
            ctx.strokeStyle = "gray";
            ctx.beginPath();
            ctx.moveTo(0, 0);
            this.x = 30 * Math.cos((this.angle * Math.PI) / 180);
            this.y = 30 * Math.sin((this.angle * Math.PI) / 180);
            ctx.lineTo(this.x, this.y);
            ctx.stroke();
            ctx.lineWidth = 3;
            ctx.lineCap = "round";
            ctx.strokeStyle = "yellow";
            ctx.beginPath();
            ctx.arc(
              this.x + this.r * Math.cos((this.angle * Math.PI) / 180),
              this.y + this.r * Math.sin((this.angle * Math.PI) / 180),
              this.r,
              ((90 + this.angle) * Math.PI) / 180,
              ((270 + this.angle) * Math.PI) / 180
            );
            ctx.stroke();
            ctx.restore();
            this.x += this.owner.x + this.owner.width / 2;
            this.y += this.owner.y + this.owner.height;
          } else if (this.status == 1) {
            is_pulling = true;
            this.x += this.moveSpeed * Math.cos((this.angle * Math.PI) / 180);
            this.y += this.moveSpeed * Math.sin((this.angle * Math.PI) / 180);
            console.log("this.x, this.y", this.x, this.y);
            ctx.beginPath();
            ctx.lineWidth = 1;
            ctx.lineCap = "round";
            ctx.strokeStyle = "gray";
            ctx.moveTo(
              this.owner.x + this.owner.width / 2,
              this.owner.y + this.owner.height
            );

            ctx.lineTo(this.x, this.y);
            ctx.stroke();
            ctx.lineWidth = 3;
            ctx.lineCap = "round";
            ctx.strokeStyle = "red";
            ctx.beginPath();
            ctx.arc(
              this.x + this.r * Math.cos((this.angle * Math.PI) / 180),
              this.y + this.r * Math.sin((this.angle * Math.PI) / 180),
              this.r,
              ((this.angle + 90) * Math.PI) / 180,
              ((this.angle - 90) * Math.PI) / 180
            );
            ctx.stroke();
          } else if (this.status == 2) {
            is_pulling = true;
            this.x += this.moveSpeed * Math.cos((this.angle * Math.PI) / 180);
            this.y += this.moveSpeed * Math.sin((this.angle * Math.PI) / 180);
            ctx.lineWidth = 1;
            ctx.lineCap = "round";
            ctx.strokeStyle = "gray";
            ctx.beginPath();
            ctx.moveTo(
              this.owner.x + this.owner.width / 2,
              this.owner.y + this.owner.height
            );
            ctx.lineTo(this.x, this.y);
            ctx.stroke();
            ctx.lineWidth = 3;
            ctx.lineCap = "round";
            ctx.strokeStyle = "green";
            ctx.beginPath();
            ctx.arc(
              this.x + this.r * Math.cos((this.angle * Math.PI) / 180),
              this.y + this.r * Math.sin((this.angle * Math.PI) / 180),
              this.r,
              ((this.angle + 90) * Math.PI) / 180,
              ((this.angle - 90) * Math.PI) / 180
            );
            ctx.stroke();
          }
        },
        check: function () {
          this.length = Math.sqrt(
            (this.x - this.owner.x) * (this.x - this.owner.x) +
              (this.y - this.owner.y) * (this.y - this.owner.y)
          );

          if (this.length >= this.maxLength && this.status == 1) {
            this.moveSpeed = -this.moveSpeed;
          }

          if (
            canvas.height - this.y > 0 &&
            canvas.height - this.y < 84 &&
            this.y < canvas.width &&
            this.status != 0 &&
            this.moveSpeed < 0
          ) {
            if (this.status == 2) {
              if (this.object.value == -1) {
                time_all.value += this.object.timeAdd;
                this.object.x = -100;
                this.object.y = -100;
                this.object.owner = 0;
                this.object.moveSpeed = 0;
                this.object = 0;
                this.status = 0;
                this.moveSpeed = 20;
              } else {
                if (mode == 0) {
                  catching_object = this;
                }
                setTimeout(this.pull_fish(), 1000);
                this.moveSpeed = 0;
              }
            } else {
              this.moveSpeed = 20;
              this.status = 0;
            }
          }
        },

        pull_fish: function () {
          if (mode == 0) {
            catching_object.object.moveSpeed = 0;
            catching_object.object.x = -100;
            catching_object.object.y = -100;
            catching_object.status = 0;
            // catching_object.object.owner.object = 0;
            catching_object.moveSpeed = 20;
            catching_object.object.owner = 0;
          }
        },
      };

      async function OpenBootstrapPopup() {
        // @ts-ignore
        // await $("#myModal").modal("show");
        clearInterval(time_limit);
      }

      // BEGIN: TigerBui`
      function getRandomArbitrary(min: number, max: number) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min + 1)) + min;
      }
      function windowWH() {
        return {
          w: window.innerWidth,
          h: window.innerHeight,
        };
      }
      // END: TigerBui`
      function init() {
        // time_all.value = 6;
        // @ts-ignore
        players.push(new player(0));
        level = 1;
        if (time_all.value > 1) {
          timeUpdate();
        }
        if (time_all.value > 1) {
          createLevel(level);
        }
      }
      init();
    }

    return {
      couponR,
      couponRN,
      couponArray,
      time_all,
    };
  },
};
</script>

<style>
#canvas1 {
  padding: 0px;
  margin: auto 0px;
  border: 4px solid black;
  background: linear-gradient(to bottom, lightblue, rgb(0, 14, 139));
}
</style>
