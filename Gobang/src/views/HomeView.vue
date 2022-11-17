<template>
    <div class="home">
        <h1>基于α-β剪枝的五子棋小游戏</h1>
        <el-form :inline="true" :model="settings" class="demo-form-inline">
            <el-form-item label="棋盘大小">
                <el-select v-model="settings.size" placeholder="棋盘大小">
                    <el-option label="11x11 (推荐)" :value="11" />
                    <el-option label="15x15 (也行)" :value="15" />
                </el-select>
            </el-form-item>
            <el-form-item label="先手">
                <el-select v-model="settings.first" placeholder="选择谁先开始">
                    <el-option label="玩家" :value="0" />
                    <el-option label="电脑" :value="1" />
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="onSubmit">开始下棋！</el-button>
                <el-button type="info" @click="onRefresh">重新开始！</el-button>
            </el-form-item>
        </el-form>
        <div class="container">
            <div v-for="x in parseInt(data)" class="row" :key="x">
                <span v-for="y in parseInt(data)" class="chess" :id="parseInt(data) * (x - 1) + y" :key="x * 100 + y"
                    @click="onClickBoard(x, y)">
                </span>
            </div>
            <div class="bg">
                <div v-for="x in parseInt(data) - 1" class="row" :key="x">
                    <span v-for="y in parseInt(data) - 1" class="bg-chess" :id="parseInt(data) * (x - 1) + y"
                        :key="x * 100 + y">
                    </span>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'


const inf = 9999999
const pattern_score = {
    // 1
    "10000": 1,
    "01000": 2,
    "00100": 3,
    "00010": 2,
    "00001": 1,
    // 2
    "11000": 60,
    "10100": 50,
    "10010": 30,
    "10001": 10,
    "01100": 100,
    "01010": 80,
    "01001": 30,
    "00110": 100,
    "00101": 50,
    "00011": 60,
    // 3
    "00111": 600,
    "01011": 600,
    "01101": 800,
    "01110": 1000,
    "10011": 400,
    "10101": 300,
    "10110": 800,
    "11001": 400,
    "11010": 600,
    "11100": 600,
    // 4
    "01111": 5000,
    "10111": 5000,
    "11011": 5000,
    "11101": 5000,
    "11110": 5000,
    // 5
    "11111": inf
}
const count_steps = ref(1)
const color_pad = ['white', 'black']
const data = ref(1)
const settings = ref({
    size: '',
    first: '',
})
const do_chess = {
    'ai': 1,
    'me': -1
}
const chess_board_arr = ref(null)
const search_field = ref({
    row_start: inf,
    row_end: -inf,
    col_start: inf,
    col_end: -inf
})

// 根据x,y下标，获取domId
const getDomId = (x, y) => { return parseInt(data.value) * (x - 1) + y }

// 开始！
const onSubmit = () => {
    // 设置棋盘大小
    data.value = parseInt(settings.value.size)
    // 创建数组
    chess_board_arr.value = null
    chess_board_arr.value = Array.from(Array(data.value), () => Array(data.value).fill(0))
    // 判断先手
    setTimeout(() => {
        if (settings.value.first === 1) {

            console.log(chess_board_arr.value);
            let [x, y] = [parseInt(data.value / 2) + 1, parseInt(data.value / 2) + 1]

            let id = getDomId(x, y)
            console.log('id', id);
            // 获取dom
            let dom = document.getElementById(id)
            // 设置属性
            dom.style.backgroundColor = color_pad[1]
            dom.innerHTML = `${count_steps.value}`
            dom.style.color = color_pad[0]
            dom.style.lineHeight = '32px'
            count_steps.value++
            // 更新矩阵
            chess_board_arr.value[x - 1][y - 1] = do_chess.ai
            // 更新搜索边界
            update_field(x - 1, y - 1)
            console.log('upd-f', search_field.value);
        }
    }, 500);
}

// 刷新
const onRefresh = () => {
    window.location.reload()
}

// 备份搜索区域
const back_field = () => {
    return [search_field.value.row_start, search_field.value.row_end, search_field.value.col_start, search_field.value.col_end]
}

// 更新搜索区域
const update_field = (x, y) => {
    search_field.value.row_start = Math.min(Math.max(x - 1, 0), search_field.value.row_start)
    search_field.value.row_end = Math.max(Math.min(x + 1, data.value - 1), search_field.value.row_end)
    search_field.value.col_start = Math.min(Math.max(y - 1, 0), search_field.value.col_start)
    search_field.value.col_end = Math.max(Math.min(y + 1, data.value - 1), search_field.value.col_end)
}

// 恢复搜索区域
const traceback_field = (rs, re, cs, ce) => {
    search_field.value.row_start = rs
    search_field.value.row_end = re
    search_field.value.col_start = cs
    search_field.value.col_end = ce
}

// 提示框
const showNotice = (msg) => {
    ElMessageBox.alert('要重新开始吗？', `${msg}`, {
        autofocus: false,
        showCancelButton: true,
        confirmButtonText: '重新开始',
        cancelButtonText: '一会再说',
        callback: (action) => {
            if (action === 'confirm') {
                onRefresh()
            }
        },
    })
}

// 下棋
const onClickBoard = (x, y) => {
    // 不能下载重复位置
    if (chess_board_arr.value[x - 1][y - 1] === 0) {
        // 获取dom的id
        let id = getDomId(x, y)
        // 获取dom
        let dom = document.getElementById(id)
        // 设置属性
        dom.style.backgroundColor = color_pad[0]
        dom.innerHTML = `${count_steps.value}`
        dom.style.color = color_pad[1]
        dom.style.lineHeight = '32px'
        // 棋子编号
        count_steps.value++
        // 更新矩阵
        chess_board_arr.value[x - 1][y - 1] = do_chess.me
        // 更新搜索边界
        update_field(x - 1, y - 1)

        if (state() !== State.on) {
            if (state() === State.ai_win) {
                showNotice('好可惜，你输了！')
            }
            else if (state() === State.human_win) {
                showNotice('好厉害，你赢了！')
            }
            else if (state() === State.tie) {
                showNotice('棋盘下满了,和局！')
            }
        }
        else {
            let [x, y, _x, _y] = work()
            let id = getDomId(x + 1, y + 1)
            let dom = document.getElementById(id)
            dom.style.backgroundColor = color_pad[1]
            dom.innerHTML = `${count_steps.value}`
            dom.style.color = color_pad[0]
            dom.style.lineHeight = '32px'
            count_steps.value++
            // 更新矩阵
            chess_board_arr.value[x][y] = do_chess.ai
            // 更新搜索边界
            update_field(x, y)
            if (state() !== State.on) {
                if (state() === State.ai_win) {
                    showNotice('好可惜，你输了！')
                }
                else if (state() === State.human_win) {
                    showNotice('好厉害，你赢了！')
                }
                else if (state() === State.tie) {
                    showNotice('棋盘下满了,和局！')
                }
                else {
                    console.log(_x, _y);
                }
            }
        }
    }
    else {
        ElMessage({
            showClose: true,
            message: '看看这儿能下吗？',
            type: 'error',
        })
    }
}

// 棋局状态
const State = {
    "ai_win": 0,
    "human_win": 1,
    "tie": 2,
    "on": 3
}

// 判断位置在不在棋盘里
const pos_in_board = (x, y) => {
    return x >= 0 && y >= 0 && x < data.value && y < data.value
}

// 判断指定位置有没有棋子
const pos_is_empty = (x, y) => {
    return chess_board_arr.value[x][y] === 0
}

// 判断棋局状态
const state = () => {
    let full = true
    for (let i = 0; i < data.value; i++) {
        for (let j = 0; j < data.value; j++) {
            // 判断是否是满的
            if (chess_board_arr.value[i][j] === 0) {
                full = false
                continue
            }
            // 5左
            if (pos_in_board(i, j + 4)) {
                let win = true
                for (let k = 0; k < 5; k++) {
                    if (chess_board_arr.value[i][j + k] !== chess_board_arr.value[i][j]) {
                        win = false
                        break
                    }
                }
                if (win) {
                    if (chess_board_arr.value[i][j] === 1) {
                        return State.ai_win
                    }
                    else {
                        return State.human_win
                    }
                }
            }
            // 5上
            if (pos_in_board(i + 4, j)) {
                let win = true
                for (let k = 0; k < 5; k++) {
                    if (chess_board_arr.value[i + k][j] !== chess_board_arr.value[i][j]) {
                        win = false
                        break
                    }
                }
                if (win) {
                    if (chess_board_arr.value[i][j] === 1) {
                        return State.ai_win
                    }
                    else {
                        return State.human_win
                    }
                }
            }
            // 5左斜上
            if (pos_in_board(i + 4, j + 4)) {
                let win = true
                for (let k = 0; k < 5; k++) {
                    if (chess_board_arr.value[i + k][j + k] !== chess_board_arr.value[i][j]) {
                        win = false
                        break
                    }
                }
                if (win) {
                    if (chess_board_arr.value[i][j] === 1) {
                        return State.ai_win
                    }
                    else {
                        return State.human_win
                    }
                }
            }
            // 5右斜上
            if (pos_in_board(i - 4, j + 4)) {
                let win = true
                for (let k = 0; k < 5; k++) {
                    if (chess_board_arr.value[i - k][j + k] !== chess_board_arr.value[i][j]) {
                        win = false
                        break
                    }
                }

                if (win) {
                    if (chess_board_arr.value[i][j] === 1) {
                        return State.ai_win
                    }
                    else {
                        return State.human_win
                    }
                }
            }
        }
    }
    if (full) {
        return State.tie
    }
    return State.on
}

// 估值函数(简易)
const evaluate = () => {
    let res = 0
    for (let i = 0; i < data.value; i++) {
        for (let j = 0; j < data.value; j++) {
            // 5左
            if (pos_in_board(i, j + 4)) {
                let pattern = []
                let m_pattern = []
                for (let k = 0; k < 5; k++) {
                    pattern.push(chess_board_arr.value[i][j + k])
                    m_pattern.push(-chess_board_arr.value[i][j + k])
                }
                let cur_pattern = pattern.toString().replaceAll(',', '')
                let cur_m_pattern = m_pattern.toString().replaceAll(',', '')
                if (pattern_score[cur_pattern]) {
                    res = res + pattern_score[cur_pattern]
                }
                if (pattern_score[cur_m_pattern]) {
                    res = res - pattern_score[cur_m_pattern]
                }
            }

            // 5上
            if (pos_in_board(i + 4, j)) {
                let pattern = []
                let m_pattern = []
                for (let k = 0; k < 5; k++) {
                    pattern.push(chess_board_arr.value[i + k][j])
                    m_pattern.push(-chess_board_arr.value[i + k][j])
                }
                let cur_pattern = pattern.toString().replaceAll(',', '')
                let cur_m_pattern = m_pattern.toString().replaceAll(',', '')
                if (pattern_score[cur_pattern]) {
                    res = res + pattern_score[cur_pattern]
                }
                if (pattern_score[cur_m_pattern]) {
                    res = res - pattern_score[cur_m_pattern]
                }
            }

            // 5左斜上
            if (pos_in_board(i + 4, j + 4)) {
                let pattern = []
                let m_pattern = []
                for (let k = 0; k < 5; k++) {
                    pattern.push(chess_board_arr.value[i + k][j + k])
                    m_pattern.push(-chess_board_arr.value[i + k][j + k])
                }
                let cur_pattern = pattern.toString().replaceAll(',', '')
                let cur_m_pattern = m_pattern.toString().replaceAll(',', '')
                if (pattern_score[cur_pattern]) {
                    res = res + pattern_score[cur_pattern]
                }
                if (pattern_score[cur_m_pattern]) {
                    res = res - pattern_score[cur_m_pattern]
                }
            }

            // 5右斜上
            if (pos_in_board(i - 4, j + 4)) {
                let pattern = []
                let m_pattern = []
                for (let k = 0; k < 5; k++) {
                    pattern.push(chess_board_arr.value[i - k][j + k])
                    m_pattern.push(-chess_board_arr.value[i - k][j + k])
                }
                let cur_pattern = pattern.toString().replaceAll(',', '')
                let cur_m_pattern = m_pattern.toString().replaceAll(',', '')
                if (pattern_score[cur_pattern]) {
                    res = res + pattern_score[cur_pattern]
                }
                if (pattern_score[cur_m_pattern]) {
                    res = res - pattern_score[cur_m_pattern]
                }
            }
        }
    }
    return res

}

// 得到下棋的位置
const work = () => {
    let [x, y, _x, _y] = max_step(-inf)
    return [x, y, _x, _y]
}

// minmax搜索, α-β剪枝
const max_step = (now) => {
    let x = 0
    let y = 0
    let _x = 0
    let _y = 0
    for (let i = search_field.value.row_start; i <= search_field.value.row_end; i++) {
        for (let j = search_field.value.col_start; j <= search_field.value.col_end; j++) {
            // 尝试下棋，查看估值 -> "访问子节点"
            if (pos_in_board(i, j) && pos_is_empty(i, j)) {
                chess_board_arr.value[i][j] = do_chess.ai
                let [a, b, c, d] = back_field()
                update_field(i, j)
                let [new_max, px, py] = min_step(now)
                if (new_max > now) {
                    x = i
                    y = j
                    _x = px
                    _y = py
                    now = new_max
                }
                // 记得恢复现场
                chess_board_arr.value[i][j] = 0
                traceback_field(a, b, c, d)
            }
        }
    }
    return [x, y, _x, _y]
}

// 极小搜索
const min_step = (now) => {
    let res = inf
    let x = 0
    let y = 0
    for (let i = search_field.value.row_start; i <= search_field.value.row_end; i++) {
        for (let j = search_field.value.col_start; j <= search_field.value.col_end; j++) {
            // 尝试下棋，查看估值 -> "访问子节点"
            if (pos_in_board(i, j) && pos_is_empty(i, j)) {
                chess_board_arr.value[i][j] = do_chess.me
                let score = evaluate()
                chess_board_arr.value[i][j] = 0
                if (score < res) {
                    res = score
                    x = i
                    y = j
                }
                if (res < now) {
                    return [now, x, y]
                }
            }
        }
    }
    return [res, x, y]
}

</script>

<style scoped>
.row {
    display: flex;
    justify-content: center;
}

.container {
    text-align: center;
    position: relative;
    background-color: burlywood;
    padding: 15px;
}

.chess {
    display: inline-block;
    width: 32px;
    height: 32px;
    border-radius: 50%;
    margin: 2px;
    z-index: 5;
}

.chess:hover {
    box-shadow: 0px 0px 5px grey;
    transform: scaleX(1.05);
    transform: scaleY(1.05);
    transition: 0.3s;
}

.bg-chess {
    display: inline-block;
    width: 34px;
    height: 34px;
    border: 1px solid;
    background-color: burlywood;
}

.bg {
    position: absolute;
    top: 34px;
    left: 0px;
    bottom: 0;
    right: 0;
    margin: auto;
}

.home {
    text-align: center;
}
</style>