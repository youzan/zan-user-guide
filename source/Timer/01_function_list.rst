Timer 定时器使用示例
=========================


1. Timer 
    具体参考 API 文档：http://zandoc.zanphp.io/namespaces/default.html
    
    swoole_timer_tick、swoole_timer_after、swoole_timer_clear、swoole_timer_exists、swoole_timer_set


2. 使用示例:
::

    <?php
    class TimerTest {
        public static $count = 0;
        private $timer_id = null;

        protected function resetTimer($ms) {
            if ($this->timer_id && swoole_timer_exists($this->timer_id)) {
                swoole_timer_clear($this->timer_id);
                $this->timer_id = null;
            }
            if (self::$count == 10) {
                return;
            }
            $this->timer_id = swoole_timer_after($ms, array($this, 'onTimerTick'));
            assert($this->timer_id > 0);
        }

        public function onTimerTick() {
            self::$count++;
            echo "onTimerTick:" . self::$count . "\n";
            $this->resetTimer(10);
        }
    }

    $timer_test = new TimerTest();
    $timer_test->onTimerTick();


::

