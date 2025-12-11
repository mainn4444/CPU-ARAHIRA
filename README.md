<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CPU動作シミュレーター</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
</head>
<body class="min-h-screen bg-gray-50 p-8">

    <div class="max-w-6xl mx-auto">
        <div class="bg-white rounded border border-gray-300 p-6 mb-6">
            <div class="flex items-center gap-3 mb-4">
                <i data-lucide="book-open" class="w-8 h-8 text-gray-700"></i>
                <h1 class="text-3xl font-bold text-gray-800">CPU動作シミュレーター</h1>
            </div>
            <p class="text-gray-600">CPUの基本的な動作(フェッチ→デコード→実行)を行おう！！</p>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <div class="bg-white rounded border border-gray-300 p-6">
                <h2 class="text-xl font-bold text-gray-800 mb-4">プログラム</h2>
                
                <div id="program-list" class="space-y-2 mb-4">
                    </div>
                
                <button onclick="addProgramLine()" id="btn-add" class="flex items-center gap-2 text-sm text-gray-600 hover:text-gray-800 disabled:text-gray-300 mb-4">
                    <i data-lucide="plus" class="w-4 h-4"></i>
                    命令を追加
                </button>
                
                <div class="mt-4 p-4 bg-gray-100 rounded border border-gray-300">
                    <h3 class="font-semibold text-sm text-gray-800 mb-2">使える命令</h3>
                    <div class="text-xs text-gray-700 space-y-1">
                        <div><code class="bg-white px-1">READ レジスタ アドレス</code> - メモリからレジスタに読み込む</div>
                        <div><code class="bg-white px-1">WRITE レジスタ アドレス</code> - レジスタの値をメモリに書き込む</div>
                        <div><code class="bg-white px-1">ADD レジスタ1 レジスタ2</code> - レジスタ1にレジスタ2を加算</div>
                        <div><code class="bg-white px-1">STOP</code> - プログラム終了</div>
                    </div>
                </div>

                <div class="flex gap-2 mt-4">
                    <button onclick="runProgram()" id="btn-run" class="flex items-center gap-2 bg-gray-800 text-white px-4 py-2 rounded hover:bg-gray-700 disabled:bg-gray-400">
                        <i data-lucide="play" class="w-4 h-4"></i>
                        実行
                    </button>
                    <button onclick="stepProgram()" id="btn-step" class="flex items-center gap-2 bg-gray-600 text-white px-4 py-2 rounded hover:bg-gray-500 disabled:bg-gray-400">
                        <i data-lucide="step-forward" class="w-4 h-4"></i>
                        1ステップ
                    </button>
                    <button onclick="resetSystem()" class="flex items-center gap-2 bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-400">
                        <i data-lucide="rotate-ccw" class="w-4 h-4"></i>
                        リセット
                    </button>
                </div>
            </div>

            <div class="space-y-6">
                <div class="bg-white rounded border border-gray-300 p-6">
                    <h2 class="text-xl font-bold text-gray-800 mb-4">レジスタ(高速記憶領域)</h2>
                    <div class="grid grid-cols-2 gap-4">
                        <div class="bg-gray-100 border border-gray-300 p-4 rounded text-center">
                            <div class="text-sm font-semibold text-gray-700">A</div>
                            <div id="reg-A" class="text-3xl font-bold text-gray-900">0</div>
                        </div>
                        <div class="bg-gray-100 border border-gray-300 p-4 rounded text-center">
                            <div class="text-sm font-semibold text-gray-700">B</div>
                            <div id="reg-B" class="text-3xl font-bold text-gray-900">0</div>
                        </div>
                    </div>
                </div>

                <div class="bg-white rounded border border-gray-300 p-6">
                    <h2 class="text-xl font-bold text-gray-800 mb-4">制御装置</h2>
                    <div class="space-y-3">
                        <div class="bg-gray-100 border border-gray-300 p-3 rounded">
                            <div class="text-sm font-semibold text-gray-700">プログラムカウンタ (PC)</div>
                            <div id="val-pc" class="text-2xl font-bold text-gray-900">0</div>
                        </div>
                        <div class="bg-gray-100 border border-gray-300 p-3 rounded">
                            <div class="text-sm font-semibold text-gray-700">命令レジスタ (IR)</div>
                            <div id="val-ir" class="text-sm font-mono text-gray-900">---</div>
                        </div>
                    </div>
                </div>

                <div class="bg-white rounded border border-gray-300 p-6">
                    <h2 class="text-xl font-bold text-gray-800 mb-4">メモリ(主記憶装置)</h2>
                    <div id="memory-grid" class="grid grid-cols-5 gap-2">
                        </div>
                </div>
            </div>
        </div>

        <div class="bg-white rounded border border-gray-300 p-6 mt-6">
            <h2 class="text-xl font-bold text-gray-800 mb-4">実行ログ</h2>
            <div id="log-area" class="bg-gray-50 border border-gray-300 p-4 rounded font-mono text-sm h-48 overflow-y-auto">
                <div class="text-gray-400">実行するとログが表示されます</div>
            </div>
        </div>
    </div>

    <script>
        // --- 状態管理 ---
        const state = {
            registers: { A: 0, B: 0 },
            memory: Array(10).fill(0),
            programCounter: 0,
            instructionRegister: '',
            isRunning: false,
            log: [],
            program: [
                { cmd: 'READ', arg1: 'A', arg2: '0' },
                { cmd: 'READ', arg1: 'B', arg2: '1' },
                { cmd: 'ADD', arg1: 'A', arg2: 'B' },
                { cmd: 'WRITE', arg1: 'A', arg2: '2' },
                { cmd: 'STOP', arg1: '', arg2: '' }
            ]
        };

        // --- 初期化 ---
        function init() {
            renderAll();
            lucide.createIcons();
        }

        // --- ロジック関数 ---
        
        function executeInstruction(instruction) {
            const { cmd, arg1, arg2 } = instruction;
            let message = '';
            let shouldContinue = true;
            
            switch(cmd) {
                case 'READ':
                    const addr = parseInt(arg2);
                    const valRead = state.memory[addr];
                    state.registers[arg1] = valRead;
                    message = `メモリ[${addr}]からレジスタ${arg1}に読み込み (値: ${valRead})`;
                    break;
                    
                case 'WRITE':
                    const writeAddr = parseInt(arg2);
                    const writeValue = state.registers[arg1];
                    state.memory[writeAddr] = writeValue;
                    message = `レジスタ${arg1}の値をメモリ[${writeAddr}]に書き込み (値: ${writeValue})`;
                    break;
                    
                case 'ADD':
                    const sum = state.registers[arg1] + state.registers[arg2];
                    state.registers[arg1] = sum;
                    message = `${arg1} = ${arg1} + ${arg2} (結果: ${sum})`;
                    break;
                    
                case 'STOP':
                    message = 'プログラム終了';
                    shouldContinue = false;
                    break;
                    
                default:
                    message = '不明な命令';
                    shouldContinue = false;
            }
            
            addLog(`PC=${state.programCounter}: ${cmd} ${arg1} ${arg2} → ${message}`);
            return shouldContinue;
        }

        function stepProgram() {
            if (state.programCounter >= state.program.length) {
                state.isRunning = false;
                addLog('プログラムの終端に到達しました');
                renderAll();
                return;
            }

            const currentInstruction = state.program[state.programCounter];
            const instructionText = `${currentInstruction.cmd} ${currentInstruction.arg1} ${currentInstruction.arg2}`.trim();
            state.instructionRegister = instructionText;
            
            // UIを一旦更新してPCの場所を示す
            renderAll();

            const shouldContinue = executeInstruction(currentInstruction);
            
            if (shouldContinue) {
                state.programCounter++;
            } else {
                state.isRunning = false;
            }
            renderAll();
        }

        function runProgram() {
            if(state.isRunning) return;
            state.isRunning = true;
            renderAll();

            function loop() {
                if (!state.isRunning) return;
                
                if (state.programCounter >= state.program.length) {
                    state.isRunning = false;
                    renderAll();
                    return;
                }

                stepProgram(); // 1ステップ実行

                // もし停止していなければ次へ
                if (state.isRunning) {
                    setTimeout(loop, 1000); // 1秒ごとに実行
                }
            }
            loop();
        }

        function resetSystem() {
            state.registers = { A: 0, B: 0 };
            state.memory = Array(10).fill(0);
            state.programCounter = 0;
            state.instructionRegister = '';
            state.isRunning = false;
            state.log = [];
            renderAll();
        }

        function addLog(text) {
            state.log.push(text);
            renderLog();
        }

        // --- プログラム編集関連 ---
        function addProgramLine() {
            state.program.push({ cmd: 'READ', arg1: 'A', arg2: '0' });
            renderProgramList();
        }

        function deleteProgramLine(index) {
            if (state.program.length > 1) {
                state.program.splice(index, 1);
                renderProgramList();
            }
        }

        function updateProgram(index, field, value) {
            state.program[index][field] = value;
            renderProgramList();
        }

        function updateMemory(index, value) {
            state.memory[index] = parseInt(value) || 0;
            // メモリ入力はフォーカスが外れないように再描画を工夫するか、
            // 単純に値を保持するだけにする。ここでは簡易的に再描画せず値だけ更新し、
            // 実行時に反映されるようにする（ただし表示同期のためrenderMemoryを呼ぶのがベター）
            // 今回はシンプルに再描画はrun/step時のみに行われるが、手動入力時はinputのvalueがそのまま残るためOK
        }

        // --- 描画関数 (UI更新) ---
        function renderAll() {
            document.getElementById('reg-A').textContent = state.registers.A;
            document.getElementById('reg-B').textContent = state.registers.B;
            document.getElementById('val-pc').textContent = state.programCounter;
            document.getElementById('val-ir').textContent = state.instructionRegister || '---';
            
            renderMemory();
            renderProgramList();
            renderLog();
            updateButtonStates();
        }

        function renderMemory() {
            const container = document.getElementById('memory-grid');
            // 編集中でなければ中身を書き換える（フォーカス問題を避けるため簡易実装）
            // ここでは毎回全書き換えします
            if (document.activeElement && document.activeElement.classList.contains('mem-input')) {
                return; // 入力中は再描画しない
            }

            container.innerHTML = state.memory.map((val, i) => `
                <div class="bg-gray-50 border border-gray-300 p-2 rounded">
                    <div class="text-xs font-semibold text-gray-600 text-center mb-1">[${i}]</div>
                    <input
                        type="number"
                        class="mem-input w-full text-center text-sm font-bold text-gray-900 border border-gray-300 rounded px-1"
                        value="${val}"
                        onchange="updateMemory(${i}, this.value)"
                        ${state.isRunning ? 'disabled' : ''}
                    />
                </div>
            `).join('');
        }

        function renderProgramList() {
            const container = document.getElementById('program-list');
            const getArg1Opts = (cmd) => ['READ', 'WRITE', 'ADD'].includes(cmd) ? ['A', 'B'] : [''];
            const getArg2Opts = (cmd) => {
                if(['READ', 'WRITE'].includes(cmd)) return [0,1,2,3,4,5,6,7,8,9];
                if(cmd === 'ADD') return ['A', 'B'];
                return [''];
            };

            container.innerHTML = state.program.map((line, i) => {
                const isCurrent = state.programCounter === i;
                const badgeClass = isCurrent ? 'bg-gray-800 text-white font-bold' : 'bg-gray-200';
                
                // コマンド選択肢
                const cmdSelect = `
                    <select onchange="updateProgram(${i}, 'cmd', this.value)" 
                            class="border border-gray-300 rounded px-2 py-1 text-sm font-mono"
                            ${state.isRunning ? 'disabled' : ''}>
                        ${['READ', 'WRITE', 'ADD', 'STOP'].map(opt => 
                            `<option value="${opt}" ${line.cmd === opt ? 'selected' : ''}>${opt}</option>`
                        ).join('')}
                    </select>
                `;

                let argsInputs = '';
                if (line.cmd !== 'STOP') {
                    argsInputs += `
                        <select onchange="updateProgram(${i}, 'arg1', this.value)"
                                class="border border-gray-300 rounded px-2 py-1 text-sm font-mono"
                                ${state.isRunning ? 'disabled' : ''}>
                            ${getArg1Opts(line.cmd).map(opt => `<option value="${opt}" ${line.arg1 == opt ? 'selected' : ''}>${opt}</option>`).join('')}
                        </select>
                        <select onchange="updateProgram(${i}, 'arg2', this.value)"
                                class="border border-gray-300 rounded px-2 py-1 text-sm font-mono"
                                ${state.isRunning ? 'disabled' : ''}>
                            ${getArg2Opts(line.cmd).map(opt => `<option value="${opt}" ${line.arg2 == opt ? 'selected' : ''}>${opt}</option>`).join('')}
                        </select>
                    `;
                }

                return `
                    <div class="flex gap-2 items-center">
                        <span class="w-8 text-center font-mono text-sm ${badgeClass} rounded px-2 py-1">${i}</span>
                        ${cmdSelect}
                        ${argsInputs}
                        <button onclick="deleteProgramLine(${i})" 
                                class="ml-auto p-1 text-gray-600 hover:text-red-600 disabled:text-gray-300"
                                ${state.isRunning || state.program.length <= 1 ? 'disabled' : ''}>
                            <i data-lucide="trash-2" class="w-4 h-4"></i>
                        </button>
                    </div>
                `;
            }).join('');
            
            lucide.createIcons(); // アイコン再生成
        }

        function renderLog() {
            const container = document.getElementById('log-area');
            if (state.log.length === 0) {
                container.innerHTML = '<div class="text-gray-400">実行するとログが表示されます</div>';
            } else {
                container.innerHTML = state.log.map(entry => `<div class="text-gray-800 mb-1">${entry}</div>`).join('');
                container.scrollTop = container.scrollHeight;
            }
        }

        function updateButtonStates() {
            document.getElementById('btn-run').disabled = state.isRunning;
            document.getElementById('btn-step').disabled = state.isRunning || state.programCounter >= state.program.length;
            document.getElementById('btn-add').disabled = state.isRunning;
        }

        // 開始
        init();

    </script>
</body>
</html>
