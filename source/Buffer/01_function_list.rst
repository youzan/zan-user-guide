swoole_buffer 使用示例
=========================

1. swoole_buffer 成员函数
    具体参考 API 文档：http://zandoc.zanphp.io/classes/swoole_buffer.html

    append，clear，expand，read，write，recycle，substr

2. 使用示例
::

    <?php
        #函数都非常简单且易理解
        $buffer = new swoole_buffer();
        assert($buffer instanceof swoole_buffer);

        $data = "Test: write to buffer something.";
        $data_len = strlen($data);

        #write to buffer
        $write_len = $buffer->write(0, $data);
        assert($data_len === $write_len);

        #read some byte
        $read_str = $buffer->read(0, $data_len);
        if (strcmp($read_str, $data) == 0) {
            echo "Read str::  " . $read_str . "\n";
        }

        #append
        $data = ":append some strings.";
        $write_len = $buffer->append($data);
        echo "After append:: " . $buffer . "\n";

        #substr
        $str = $buffer->substr(0);
        echo "substr:: " . $str . "\n";

        #expand
        $new_size = 256;
        $expand_ret = $buffer->expand($new_size);

        #recycl
        $length = $buffer->length;
        $str = $buffer->substr(0, $length, true);
        $buffer->recycle();
        echo "After recycle:: " . $buffer->length . "\n";

        #append
        $data = ":append some strings.";
        $write_len = $buffer->append($data);
        echo "Buffer:: " . $buffer . "\n";

        #clear
        $buffer->clear();
        echo "After clear:: " . $buffer->length . "\n";