# 中国行政区划代码数据库

一个完整、准确的中国行政区划代码数据库，包含全国所有省、市、县、区、县级市（2800+ 个）。

## 📊 数据说明

- **数据来源**：中华人民共和国民政部
- **覆盖范围**：全国 34 个省级行政区、333 个地级行政区、2843 个县级行政区
- **数据格式**：Python 字典，键为 6 位行政区划代码，值为对应的行政区划名称
- **更新日期**：2026 年 3 月

## 📦 安装使用

### 方法一：直接复制文件

将 `region_codes_complete.py` 文件复制到您的项目中，然后导入使用：

```python
from region_codes_complete import REGION_CODES

# 使用示例
print(REGION_CODES["110000"])  # 输出：北京市
print(REGION_CODES["310000"])  # 输出：上海市
```

### 方法二：作为包导入

```bash
# 克隆或下载本仓库
git clone https://github.com/Albert-zhw/region-codes-lib.git
```

然后在项目中使用：

```python
import sys
sys.path.append('path/to/region-codes-lib')
from region_codes_complete import REGION_CODES
```

## 💡 使用示例

### 1. 基础查询

```python
from region_codes_complete import REGION_CODES

# 查询省级行政区
print(REGION_CODES["110000"])  # 北京市
print(REGION_CODES["310000"])  # 上海市
print(REGION_CODES["440000"])  # 广东省

# 查询地级市
print(REGION_CODES["440100"])  # 广州市
print(REGION_CODES["440300"])  # 深圳市

# 查询区县
print(REGION_CODES["440103"])  # 广州市荔湾区
print(REGION_CODES["440304"])  # 深圳市福田区
```

### 2. 简单查询函数

```python
from region_codes_complete import REGION_CODES

def query_region(code):
    """
    根据行政区划代码查询地名
    
    Args:
        code: 6 位行政区划代码
        
    Returns:
        str: 地名，如果未找到则返回 None
    """
    return REGION_CODES.get(code)

# 使用示例
print(query_region("110000"))  # 输出：北京市
print(query_region("440103"))  # 输出：广州市荔湾区
print(query_region("310115"))  # 输出：上海市浦东新区
```

### 3. 按前缀筛选

```python
from region_codes_complete import REGION_CODES

# 获取所有省级行政区
provinces = {code: name for code, name in REGION_CODES.items() if code.endswith('0000')}
print(f"共有 {len(provinces)} 个省级行政区")

# 获取广东省所有行政区
guangdong = {code: name for code, name in REGION_CODES.items() if code.startswith('44')}
print(f"广东省共有 {len(guangdong)} 个行政区")

# 获取广州市所有区
guangzhou = {code: name for code, name in REGION_CODES.items() if code.startswith('4401')}
print(f"广州市共有 {len(guangzhou)} 个区")
```

### 4. 反向查询（根据名称查代码）

```python
from region_codes_complete import REGION_CODES

# 创建反向索引
CODE_TO_NAME = REGION_CODES
NAME_TO_CODE = {name: code for code, name in REGION_CODES.items()}

# 根据名称查询代码
print(NAME_TO_CODE["北京市"])  # 输出：110000
print(NAME_TO_CODE["广州市"])  # 输出：440100
print(NAME_TO_CODE["广州市荔湾区"])  # 输出：440103
```

### 5. 层级查询

```python
from region_codes_complete import REGION_CODES

def get_province(code):
    """获取省份代码"""
    return code[:2] + "0000"

def get_city(code):
    """获取城市代码"""
    return code[:4] + "00"

def get_children_regions(parent_code, level="city"):
    """
    获取下级行政区
    
    Args:
        parent_code: 上级行政区代码
        level: 查询级别 ("province" 或 "city")
    
    Returns:
        下级行政区字典
    """
    if level == "province":
        # 获取该省所有城市
        prefix = parent_code[:2]
        return {code: name for code, name in REGION_CODES.items() 
                if code.startswith(prefix) and code.endswith('00') and len(code) == 6}
    elif level == "city":
        # 获取该市所有区县
        prefix = parent_code[:4]
        return {code: name for code, name in REGION_CODES.items() 
                if code.startswith(prefix) and not code.endswith('00')}
    
# 示例
print(get_province("440103"))  # 输出：440000
print(get_city("440103"))      # 输出：440100
print(get_children_regions("440000", "province"))  # 广东省所有城市
print(get_children_regions("440100", "city"))      # 广州市所有区
```

### 6. 数据验证

```python
from region_codes_complete import REGION_CODES

def is_valid_region_code(code):
    """
    验证行政区划代码是否有效
    
    Args:
        code: 6 位行政区划代码
    
    Returns:
        bool: 代码是否有效
    """
    if not code or len(code) != 6:
        return False
    return code in REGION_CODES

def get_region_name(code):
    """
    根据代码获取行政区名称
    
    Args:
        code: 6 位行政区划代码
    
    Returns:
        str: 行政区名称，无效代码返回 None
    """
    return REGION_CODES.get(code)

# 示例
print(is_valid_region_code("110000"))  # True
print(is_valid_region_code("999999"))  # False
print(get_region_name("440103"))       # 广州市荔湾区
print(get_region_name("000000"))       # None
```

### 7. 在 Web 项目中使用（Flask 示例）

```python
from flask import Flask, request, jsonify
from region_codes_complete import REGION_CODES

app = Flask(__name__)

@app.route('/api/region/<code>')
def get_region(code):
    """根据代码查询行政区"""
    region_name = REGION_CODES.get(code)
    if region_name:
        return jsonify({'code': code, 'name': region_name})
    return jsonify({'error': 'Invalid region code'}), 404

@app.route('/api/regions')
def list_regions():
    """列出所有行政区（支持过滤）"""
    prefix = request.args.get('prefix', '')
    if prefix:
        filtered = {code: name for code, name in REGION_CODES.items() 
                   if code.startswith(prefix)}
        return jsonify(filtered)
    return jsonify(REGION_CODES)

if __name__ == '__main__':
    app.run(debug=True)
```

## 📈 数据统计

```python
from region_codes_complete import REGION_CODES

# 统计各级行政区数量
provinces = [code for code in REGION_CODES if code.endswith('0000')]
cities = [code for code in REGION_CODES if code.endswith('00') and not code.endswith('0000')]
counties = [code for code in REGION_CODES if not code.endswith('00')]

print(f"省级行政区：{len(provinces)} 个")
print(f"地级行政区：{len(cities)} 个")
print(f"县级行政区：{len(counties)} 个")
print(f"总计：{len(REGION_CODES)} 个")
```

## 🔍 数据结构

行政区划代码采用 6 位数字编码：
- 前 2 位：省级行政区代码
- 中间 2 位：地级行政区代码
- 后 2 位：县级行政区代码

示例：
- `110000` - 北京市（省级）
- `110100` - 北京市市辖区（地级）
- `110101` - 北京市东城区（县级）

## ⚠️ 注意事项

1. 本数据仅供参考，具体行政区划以民政部官方公布为准
2. 行政区划可能会进行调整，请定期更新数据
3. 部分历史代码可能已变更，使用时请注意时效性
4. 特别行政区（香港、澳门、台湾）未包含在本数据库中

## 📝 许可证

本数据来源于公开渠道，可供学习、研究和使用。如需商业使用，请确保符合相关法律法规。

## 🔗 相关链接

- [中华人民共和国民政部](http://www.mca.gov.cn)
