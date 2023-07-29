<template>
  <div class="home">
    <h1>This is about page</h1>
    <div class="logs">
      <button class="glow-on-hover" style="margin-right: 15px;" ref="doButton" @click="tracingWork">GO</button>
      <button class="glow-on-hover" ref="cancelButton" @click="tracingCancel">STOP</button>
    </div>
  </div>
</template>

<script lang="ts">
import HelloWorld from '@/components/HelloWorld.vue'; // @ is an alias to /src

import {Options, Vue} from 'vue-class-component';
import Immutable from 'immutable'
import {
  interval,
  from,
  of,
  catchError,
  Observable,
  fromEvent,
  startWith,
  switchMap,
  EMPTY, Subscription
} from 'rxjs'
import {map, concatMap, delay,} from 'rxjs/operators'

function print(text: string): void {
  console.log(text)
}

@Options({
  components: {
    HelloWorld,
  },
  beforeRouteLeave(to, next) {
    this.tracingCancel()
  },
  methods: {
    tracingWork(): void {
      this.$refs.doButton.disabled = true
      this.working = true
      this.resetAbortController()
      const visibilityChange$ = fromEvent(document, 'visibilitychange');
      const constantMinutes = 8
      const constantValue = of(constantMinutes);
      let intervalArray = [1, 2, 4];


      const progressiveThread$ = visibilityChange$.pipe(
          startWith({ target: document }), // Начальное значение видимости
          switchMap((event) => {
            const isVisible = document.visibilityState === 'visible';
            if (isVisible) {
              print(`Page is visible. Working Optimized method`)
              if (this.lastSavedState) {
                const startIndex = intervalArray.indexOf(this.lastSavedState)
                intervalArray = intervalArray.slice(startIndex)
                console.log(intervalArray)
              }

              return from(intervalArray).pipe(
                  concatMap((value: number, index: number) => {
                    this.lastSavedState = value;

                    return of(value).pipe(
                        delay(value * 1000),
                        concatMap(() => this.fetchAndHandleResult(value)),
                        concatMap((result) => {
                          if (index === intervalArray.length - 1) {
                            print('[PROGRESSIVE WORK] Infinite loop started');
                            return this.runInfiniteLoop(value);
                          } else {
                            return of(result);
                          }
                        })
                    );
                  })
              );
            } else {
              this.abortController.abort()
              this.resetAbortController()
              return EMPTY; // Пустой поток, если документ скрыт
            }
          }),
      );

      const constThread$ = visibilityChange$.pipe(
          switchMap((event) => {
            const isVisible = document.visibilityState === 'visible';
            if (!isVisible) {
              print(`Page is hidden. Working constant method`)
              return interval(constantMinutes * 1000).pipe(
                  concatMap(() => {
                    return this.fetchAndHandleResult(constantMinutes);
                  })
              );
            } else {
              this.abortController.abort()
              this.resetAbortController()
              return EMPTY; // Пустой поток, если документ активен
            }
          })
      );

      this.progressiveSub = this.createSubscription$(progressiveThread$, 'PROGRESSIVE');
      this.constantSub = this.createSubscription$(constThread$, 'CONSTANT');
    },

    createSubscription$(stream$: Observable<any>, prefix:string) {
      return stream$.subscribe({
        next(val) {
          print(`[${prefix} WORK] After fetchAsync was taken = ${val}`);
        },
        error(err) {
          print(`[${prefix} WORK] Err handled ${err}`);
        },
        complete() {
          print(`[${prefix} WORK] Complete`);
        },
      });
    },

    tracingCancel(): void {
      if (this.working) {
        print('[TRACING CANCEL] Cancel')
        this.working = false
        this.$refs.doButton.disabled = false
        this.abortController.abort()
      }
    },

    fetchAndHandleResult(value: number): Observable<any> {
      const startTime = Date.now();
      return from(this.fetchAsync(this.abortSignal)).pipe(
          map((result) => {
            const endTime = Date.now();
            const duration = endTime - startTime;
            print(`[TRACING WORK] On ${value} sec completed fetch and lasted - ${duration} ms`);
            return result;
          }),
          catchError((error) => {
            let endTime = Date.now();
            let duration = endTime - startTime;
            print(`[TRACING WORK] CANCELLED on ${value} sec, fetch worked ${duration} ms`);
            throw error;
          }),
      );
    },

    runInfiniteLoop(value: number): Observable<any> {
      return interval(value * 1000).pipe(
          concatMap(() => this.fetchAndHandleResult(value))
      );
    },

    async fetchAsync(_abort: AbortSignal): Promise<number> {
      const jitterMs: number = (Math.random() * 1000) % 5000;
      const randomValue: number = Math.floor((Math.random() * 5000) % 5000)

      await this.sleepAsync(2000 + jitterMs, _abort);

      this.logs = this.logs.push(randomValue)
      print(`[FETCH ASYNC] returning ${randomValue} and currentState is ${this.logs}`)
      return randomValue;
    },

    resetAbortController() {
      this.abortController.abort();
      this.abortController = new AbortController();
      this.abortSignal = this.abortController.signal;
    },

    sleepAsync(_ms: number, _signal?: AbortSignal) {
      return new Promise<void>((_resolve, _reject) => {
        if (_signal?.aborted === true) {
          _reject(new DOMException('Operation cancelled', 'AbortError'));
          return;
        }

        let timerId: number | null = null;
        let completed = false;

        if (_signal) {
          _signal.addEventListener('abort', _event => {
            if (!completed) {
              if (timerId) {
                clearTimeout(timerId);
                timerId = null;
              }
              completed = true;
              _reject(new DOMException('Operation cancelled', 'AbortError'));
            }
          });
        }

        timerId = setTimeout(() => {
          if (completed)
            return;
          timerId = null;
          completed = true;
          _resolve();
        }, _ms);
      });
    }
  },
  data() {
    return {
      logs: Immutable.List(),
      working: false as boolean,
      abortController: new AbortController() as AbortController,
      abortSignal: null as unknown as AbortSignal,
      lastSavedState: null as unknown as number,

      progressiveSub: null as unknown as Subscription,
      constantSub: null as unknown as Subscription,
    }
  },
  created() {
    this.abortSignal = this.abortController.signal
  }
})
export default class HomeView extends Vue {
}

</script>
<style>
.home {
  margin: 10px;
  height: 400px;
  border: 1px solid black;
}

.logs {
  text-align: left;
  padding-left: 50px;
}

/*K R A C I V O*/
.glow-on-hover {
  width: 110px;
  height: 25px;
  border: none;
  outline: none;
  color: #fff;
  background: #111;
  cursor: pointer;
  position: relative;
  z-index: 0;
  border-radius: 10px;
}

.glow-on-hover:before {
  content: '';
  background: linear-gradient(45deg, #ff0000, #ff7300, #fffb00, #48ff00, #00ffd5, #002bff, #7a00ff, #ff00c8, #ff0000);
  position: absolute;
  top: -2px;
  left: -2px;
  background-size: 400%;
  z-index: -1;
  filter: blur(5px);
  width: calc(100% + 4px);
  height: calc(100% + 4px);
  animation: glowing 20s linear infinite;
  opacity: 0;
  transition: opacity .3s ease-in-out;
  border-radius: 10px;
}

.glow-on-hover:active {
  color: #000
}

.glow-on-hover:active:after {
  background: transparent;
}

.glow-on-hover:hover:before {
  opacity: 1;
}

.glow-on-hover:after {
  z-index: -1;
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: #111;
  left: 0;
  top: 0;
  border-radius: 10px;
}

@keyframes glowing {
  0% {
    background-position: 0 0;
  }
  50% {
    background-position: 400% 0;
  }
  100% {
    background-position: 0 0;
  }
}
.glow-on-hover:disabled {
  color: gray;
  cursor: not-allowed;
}

.glow-on-hover:disabled:hover:before {
  opacity: 0; /* Убираем эффект свечения */
}
</style>
