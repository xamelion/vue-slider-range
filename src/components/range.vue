<template>
 <div class="range">
   <div class="min-button" @click="changeValue(config.min.toString())">Min {{config.min | format}}</div>
   <div class="input-wrapper">
     <div class="input">
       <input type="string" @keyup="changeValue($event.target.value)" :value="value | separator" />
     </div>
     <div class="separators" ref="separators">
      <div class="separators-wrapper" ref="wrapper">
         <div class="select" ref="select"></div>
      </div>
       <ul>
         <li
          v-for="tick in config.ticks"
          :style="{ left: getElementPosition(tick) }"
          :key="tick"
        >{{tick | format}}</li>
       </ul>
     </div>
   </div>
   <div class="max-button" @click="changeValue(config.max.toString())">Max {{config.max | format}}</div>
 </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import { fromEvent, switchMap } from 'rxjs';
import { tap, takeUntil } from 'rxjs/operators';

interface IConfig {
  min: number;
  max: number;
  tick: number[];
}

Vue.filter('separator', (val:number) => `${val}`.replace(/(\d)(?=(\d{3})+([^\d]|$))/g, '$1 '));
Vue.filter('format', (val:number) => `${val}`.replace(/(\d+)\d{3}/g, '$1'+'k'));

@Component
export default class Range extends Vue {
  @Prop() private config!: IConfig;

  private value = this.config.min;

  mounted(): void {
    const select = this.$refs.select as HTMLInputElement;
    const wrapper = this.$refs.wrapper as HTMLInputElement;

    fromEvent(select,'mousedown')
    .pipe(
      switchMap((mouseDown: any) => {
        return fromEvent(document,'mousemove').pipe(
          tap((mouseMove: any) => {
            const offsetX = mouseMove.x - mouseDown.offsetX - 80;
            const wrapperWidth = wrapper.clientWidth;
            const percent = offsetX/(wrapperWidth/100);

            if(percent < 0 || percent > 100) { return; }

            this.value = this.getElementSize(percent);

            select.style.left = `${percent}%`;
          }),
          takeUntil(fromEvent(document,'mouseup')),
        );
      }),
    )
    .subscribe();
  }

  public getElementPosition(number: number): string {
    const onePercent = (this.config.max - this.config.min) / 100;
    const changePercent = Math.ceil((number - this.config.min)  / onePercent)
    return `${changePercent > 0 ? changePercent : 0}%`;
  }


  private getElementSize(value: number): number {
    const percent = (this.config.max - this.config.min) / 100;
    return Math.ceil(this.config.min + (percent * value));
  }

  private changeValue(value: string) {
    this.value = parseInt(value.replace(/\s+/g, ''), 10);

    const leftPosition = this.getElementPosition(this.value);
    const select = this.$refs.select as HTMLInputElement;

    select.style.left = leftPosition;
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  .range {
    height: 40px;
    display: flex;
    flex-direction: row;
    justify-content: space-between;

    .min-button,
    .max-button {
      border: 1px solid #5aa6ff;
      padding: 10px;
      cursor: pointer;
      color: #5aa6ff;
      font-family: 'Courier New', Courier, monospace;
      text-transform: uppercase;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      height: 16px;
      user-select: none;
      background-color: #fff;
    }

    .min-button {
      border-radius: 5px 0 0 5px;
    }

    .max-button {
      border-radius: 0 5px 5px 0;
    }

    .input-wrapper {
      flex: 1;
      .input {
        border: 1px solid #ced2d4;
        background-color: #fff;
        input {
          border-width: 0;
          text-transform: uppercase;
          height: 100%;
          width: 100%;
          outline-width: 0;
          margin: 0;
          padding: 10px;
          box-sizing: border-box;
        }
      }

      .separators {
        margin: 0 15px 0 -15px ;
        .separators-wrapper {
          margin: 0 -15px 0 15px ;
          position: relative;
           .select {
              width: 15px;
              height: 15px;
              background-color: #fff;
              border-radius: 50%;
              border: 1px solid #ced2d4;
              position: absolute;
              z-index: 2;
              margin-top: -10px;
              margin-left: -10px;
              cursor: pointer;
            }
        }
        ul {
          list-style-type: none;
          margin: 0;
          padding: 0;
          display: block;
          position: relative;
          li {
            font-size: 12px;
            color: #AEB3BA;
            position: absolute;
            text-transform: uppercase;
            display: flex;
            flex-direction: column;
            font-family: 'Courier New', Courier, monospace;
            justify-content: center;
            align-items: center;
            min-width: 30px;
            user-select: none;
            &::before {
              content: '|';
              top: 0px;
            }
          }
        }
      }
    }
  }
</style>
