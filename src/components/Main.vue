<template>
    <div>
        <div class="main" @mouseup="mouseup()">
            <div class="left">
                <ul>
                    <li v-for="name in tree" :key="name" @drag="drag(name)"><a href="#">{{name}}</a></li>
                </ul>
            </div>
            <div class="right table">
                <table>
                    <tr>
                        <th/>
                        <th :colspan="inputCol" :style="{ border: mouseover ? 'none':'black' }"
                            @mouseover="mouseenter()">Input
                        </th>
                        <th :colspan="outputCol">Output</th>
                    </tr>
                    <tr>
                        <td>
                            <button @click="addLogic()">+</button>
                        </td>

                        <td @mouseover="mouseenter()" v-for="(input, index) in inputVariablesList" :key="input">
                            <span>{{input}}</span>
                            <button @click="remove(index)">-</button>
                        </td>

                        <td v-for="output in outputVariablesList" :key="output">{{output}}</td>
                    </tr>
                    <tr v-for="(logicItem, rowIndex) in logic" :key="rowIndex">
                        <td>
                            <button @click="addLogic()">+</button>
                            <button @click="deleteLogic(rowIndex)">-</button>
                        </td>
                        <td v-for="(logicUnit, colIndex) in logicItem" :key="colIndex">
                            <select v-model="logicUnit.option">
                                <option value="=">=</option>
                                <option value="<">&#60;</option>
                                <option value=">">&#62;</option>
                            </select>
                            <input type="text" v-model="logicUnit.value"/>
                        </td>
                        <td v-for="output in outputVariablesList" :key="output">
                            {{output}}
                        </td>
                    </tr>
                </table>
            </div>

        </div>
        <hr/>
        <div class="formula">
            <h3>{{formula}}</h3>
        </div>
    </div>

</template>

<script>
    export default {
        name: "DMN-table",
        data() {
            return {
                tree: [
                    "test1",
                    "test2",
                    "test3"
                ],
                mouseover: false,
                selectName: null,
                inputVariablesList: [
                    "input_1",
                    "input_2"
                ],
                outputVariablesList: [
                    "output"
                ],
                logic: [[
                    {
                        option: ">",
                        value: "1"
                    },
                    {
                        option: "=",
                        value: "2"
                    }
                ]]
            }
        },
        computed: {
            inputCol() {
                return this.inputVariablesList && this.inputVariablesList.length || 0
            },
            outputCol() {
                return this.outputVariablesList && this.outputVariablesList.length || 0
            },
            formula() {
                let result = "";
                for (const item of this.logic) {
                    let itemLogic = "";
                    for (let index = 0; index < item.length; index++) {
                        if (item[index].option !== null && item[index].value !== null) {
                            const logic = `(\${${this.inputVariablesList[index]}}${item[index].option}${item[index].value})`;
                            itemLogic += logic;
                            if (index !== item.length - 1) {
                                itemLogic += "&&"
                            }
                        }
                    }
                    if (itemLogic.endsWith("&&")) {
                        itemLogic = itemLogic.slice(0, -2);
                    }
                    if (itemLogic) {
                        itemLogic = `(${itemLogic})||`;
                        result += itemLogic;
                    }
                }
                if (result === "()") {
                    result = null
                } else if (result.endsWith("()")) {
                    result = result.slice(0, -4);
                } else if (result.endsWith("||")) {
                    result = result.slice(0, -2);
                }
                return result;
            }
        },
        methods: {
            drag(name) {
                this.selectName = name;
            },
            mouseup() {
                this.selectName = null;
            },
            mouseenter() {
                if (this.selectName && !this.inputVariablesList.includes(this.selectName)) {
                    this.inputVariablesList.push(this.selectName);
                    this.selectName = null;
                    for (let colIndex = 0; colIndex < this.logic.length; colIndex++) {
                        this.logic[colIndex].push({
                            option: null,
                            value: null
                        })
                    }
                }
            },
            remove(index) {
                this.inputVariablesList.splice(index, 1);
                for (let rowIndex = 0; rowIndex < this.logic.length; rowIndex++) {
                    this.logic[rowIndex].splice(index, 1);
                }
            },
            addLogic() {
                let newRow = [];
                // eslint-disable-next-line no-unused-vars
                for (const item of this.inputVariablesList) {
                    newRow.push({
                        option: null,
                        value: null
                    })
                }
                this.logic.push(newRow)
            },
            deleteLogic(rowIndex) {
                this.logic.splice(rowIndex, 1);
            }
        }
    }
</script>

<style scoped>
    .main {
        display: flex;
    }

    .left {
        width: 30%;
    }

    li {
        margin: 12px;
    }

    .right {
        width: 70%
    }

    table {
        width: 100%;
    }

    td {
        padding: 12px;
    }

    th {
        padding: 16px;
    }

    .table table {
        background: black;
    }

    .table table td {
        background: white;
    }

    .table table th {
        background: white;
    }

    .formula {
        display: block;
        width: 100%;
        font-weight: bold;
        text-align: center;
    }
</style>