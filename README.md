# Ai-tech-platform-
<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>智膳AI - 剩菜逆向食谱生成器</title>
    <style>
        :root {
            --primary: #2e7d32;
            --bg: #f4f7f6;
            --text: #333;
            --danger: #d32f2f;
            --warning: #f57c00;
            --success: #388e3c;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 900px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        h1, h2 { color: var(--primary); }
        .form-group { margin-bottom: 15px; }
        label { display: block; font-weight: bold; margin-bottom: 5px; }
        input[type="text"], select {
            width: 100%; padding: 10px; border: 1px solid #ccc;
            border-radius: 6px; box-sizing: border-box;
        }
        .checkbox-group { margin: 10px 0; padding: 10px; background: #fafafa; border: 1px solid #eee; border-radius: 6px; }
        .checkbox-group label { font-weight: normal; display: flex; align-items: center; margin-bottom: 8px; }
        input[type="checkbox"] { margin-right: 10px; transform: scale(1.2); }
        
        button {
            background-color: var(--primary); color: white; border: none;
            padding: 12px 20px; border-radius: 6px; cursor: pointer;
            font-size: 16px; font-weight: bold; width: 100%; transition: 0.3s;
        }
        button:hover { background-color: #1b5e20; }
        button.danger-btn { background-color: var(--danger); margin-top: 10px; }

        /* 隐藏主页面，默认显示注册页 */
        #app-page { display: none; }

        /* 主页布局 */
        .layout { display: flex; gap: 20px; }
        .sidebar { width: 30%; background: #e8f5e9; padding: 20px; border-radius: 8px; }
        .main-content { width: 70%; }
        
        .sidebar h3 { margin-top: 0; font-size: 18px; }
        .sidebar p { margin: 8px 0; font-size: 14px; }

        /* 提示框 */
        .alert { padding: 15px; margin-top: 15px; border-radius: 6px; font-weight: bold; display: none; }
        .alert-red { background-color: #ffebee; color: var(--danger); border: 1px solid #ffcdd2; }
        .alert-yellow { background-color: #fff3e0; color: var(--warning); border: 1px solid #ffe0b2; }
        .alert-green { background-color: #e8f5e9; color: var(--success); border: 1px solid #c8e6c9; }
        
        .recipe-box { margin-top: 15px; padding: 15px; background: #fff; border: 1px dashed #ccc; border-radius: 6px; display: none; }

        /* 演示作弊器提示 */
        .demo-tip { margin-top: 20px; font-size: 13px; color: #888; border-top: 1px solid #eee; padding-top: 10px;}
    </style>
</head>
<body>

    <!-- 注册与同意页面 -->
    <div id="register-page" class="container">
        <h1>🥦 欢迎来到 智膳AI</h1>
        <p>第一步：建立你的专属饮食档案</p>

        <div class="form-group">
            <label>1. 你的厨艺程度</label>
            <select id="reg-skill">
                <option value="零基础小白">零基础小白</option>
                <option value="勉强能熟">勉强能熟</option>
                <option value="熟练大厨">熟练大厨</option>
            </select>
        </div>

        <div class="form-group">
            <label>2. 常用烹饪时间</label>
            <select id="reg-time">
                <option value="15分钟内">15分钟内 (极速)</option>
                <option value="30分钟内">30分钟内</option>
                <option value="不限">不限</option>
            </select>
        </div>

        <div class="form-group">
            <label>3. 健康及特殊饮食设定 (非常重要)</label>
            <p style="font-size:12px; color:#555;">智膳AI不会诊断疾病。请只填写你已知或经专业人士确认的饮食限制。</p>
            <select id="reg-health" multiple style="height: 100px;">
                <option value="无">沒有特別需要</option>
                <option value="G6PD">G6PD缺乏症／蠶豆症</option>
                <option value="过敏">食物過敏</option>
                <option value="素食">素食</option>
            </select>
            <small style="color:#888;">*按住 Ctrl (Mac: Cmd) 键可多选</small>
        </div>

        <div class="checkbox-group">
            <label>4. 隐私同意 (Granular Consent)</label>
            <p style="font-size:12px; color:#555;">由于健康资料属敏感资料，请主动选择授权范围：</p>
            <label><input type="checkbox" id="consent-basic" checked> 【必要】同意使用资料为本次操作提供食谱推荐。</label>
            <label><input type="checkbox" id="consent-store"> 【可选】同意将饮食偏好储存于账号，用于下次推荐。</label>
            <label><input type="checkbox" id="consent-health"> 【可选】同意储存特殊饮食设定，并在每次生成前自动套用。</label>
        </div>

        <button onclick="handleRegister()">同意并建立个人饮食档案</button>
    </div>

    <!-- 主应用页面 -->
    <div id="app-page" class="container">
        <div class="layout">
            <!-- 左侧档案 -->
            <div class="sidebar">
                <h3>👤 我的饮食档案</h3>
                <div id="profile-info"></div>
                <hr style="border:0; border-top:1px solid #ccc; margin: 15px 0;">
                <p><strong>资料控制：</strong></p>
                <button class="danger-btn" onclick="deleteHealthData()" style="font-size: 14px; padding: 8px;">删除健康资料 (模拟)</button>
            </div>

            <!-- 右侧操作区 -->
            <div class="main-content">
                <h2>🍳 今天想怎样寻找食谱？</h2>
                <p>请告诉我你冰箱里剩下的食材：</p>
                <input type="text" id="ingredients-input" placeholder="例如：鸡蛋, 洋葱, 蚕豆, 酱油">
                <button onclick="generateRecipe()" style="margin-top: 15px;">✨ 生成逆向食谱</button>

                <!-- 提示与结果区 -->
                <div id="alert-box" class="alert"></div>
                <div id="recipe-output" class="recipe-box"></div>

                <!-- 演示作弊器提示 -->
                <div class="demo-tip">
                    💡 <strong>演讲演示提示：</strong><br>
                    1. 输入 <code>鸡蛋, 洋葱</code> 触发【🟢绿灯安全】<br>
                    2. 输入 <code>鸡蛋, 蚕豆</code> 触发【🔴红灯停止】<br>
                    3. 输入 <code>鸡蛋, 酱油</code> 触发【🟡黄灯警告】
                </div>
            </div>
        </div>
    </div>

    <script>
        // 模拟数据库
        let userProfile = {};

        // 注册处理逻辑
        function handleRegister() {
            const basicConsent = document.getElementById('consent-basic').checked;
            if (!basicConsent) {
                alert("必须同意基础使用条款才能使用服务。");
                return;
            }

            // 获取多选框的值
            const healthSelect = document.getElementById('reg-health');
            const healthLimits = Array.from(healthSelect.selectedOptions).map(option => option.value);

            // 保存数据
            userProfile = {
                skill: document.getElementById('reg-skill').value,
                time: document.getElementById('reg-time').value,
                health: healthLimits
            };

            // 切换页面
            document.getElementById('register-page').style.display = 'none';
            document.getElementById('app-page').style.display = 'block';
            
            updateSidebar();
        }

        // 更新侧边栏
        function updateSidebar() {
            const isG6PD = userProfile.health.includes('G6PD');
            document.getElementById('profile-info').innerHTML = `
                <p><strong>G6PD模式：</strong> <span style="color:${isG6PD ? 'var(--success)' : '#888'}">${isG6PD ? '已启用' : '未启用'}</span></p>
                <p><strong>厨艺程度：</strong> ${userProfile.skill}</p>
                <p><strong>烹饪时间：</strong> ${userProfile.time}</p>
            `;
        }

        // 删除健康数据 (模拟资料控制)
        function deleteHealthData() {
            userProfile.health = [];
            updateSidebar();
            alert("健康资料已删除！G6PD模式已关闭。");
        }

        // 核心生成食谱逻辑
        function generateRecipe() {
            const input = document.getElementById('ingredients-input').value;
            if (!input.trim()) {
                alert("请输入食材！");
                return;
            }

            const ingredients = input.split(',').map(s => s.trim());
            const alertBox = document.getElementById('alert-box');
            const recipeBox = document.getElementById('recipe-output');
            
            alertBox.style.display = 'none';
            recipeBox.style.display = 'none';

            let status = 'GREEN'; 
            let message = '';

            // G6PD 安全护栏逻辑
            if (userProfile.health.includes('G6PD')) {
                const dangerous = ["蚕豆", "粉丝", "杂锦豆"];
                const warningItems = ["酱", "粉", "调料"];

                for (let item of ingredients) {
                    // 红灯
                    if (dangerous.some(d => item.includes(d))) {
                        status = 'RED';
                        message = `⚠️【停止推荐】发现高风险成分：${item}。因您有G6PD缺乏症（蚕豆症），系统已拦截此食谱。`;
                        break;
                    }
                    // 黄灯
                    if (warningItems.some(w => item.includes(w))) {
                        status = 'YELLOW';
                        message = `⚠️【请核对标签】食材“${item}”可能含有未标明成分，请检查包装是否含蚕豆成分。`;
                    }
                }
            }

            // 展示提示框
            alertBox.style.display = 'block';
            if (status === 'RED') {
                alertBox.className = 'alert alert-red';
                alertBox.innerHTML = message;
                return; // 红灯直接终止，不生成食谱
            } else if (status === 'YELLOW') {
                alertBox.className = 'alert alert-yellow';
                alertBox.innerHTML = message + "<br><br>（系统已生成食谱，请核对标签后食用）";
            } else {
                alertBox.className = 'alert alert-green';
                alertBox.innerHTML = "✅【根据已有资料未见限制成分】但在烹饪前请再次确认食材包装。";
            }

            // 生成食谱输出
            recipeBox.style.display = 'block';
            recipeBox.innerHTML = `
                <h3 style="margin-top:0;">🍳 专属定制食谱：${ingredients.join(' + ')} 大乱炖</h3>
                <p><strong>基于你的厨艺水平：</strong>${userProfile.skill}</p>
                <p><strong>烹饪时间：</strong>${userProfile.time}</p>
                <hr>
                <h4>👨‍🍳 步骤：</h4>
                <ol>
                    <li>将所有食材洗净切块。</li>
                    <li>根据你的常用设备（微波炉/电磁炉）进行加热。</li>
                    <li>调味出锅，注意生熟分开。</li>
                </ol>
            `;
        }
    </script>
</body>
</html>
