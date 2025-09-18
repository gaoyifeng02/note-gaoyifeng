GET - 获取资源（幂等）
  • 用于查询和检索数据

  • 示例：GET /class/students?name="Jake"

  POST - 创建资源（非幂等）
  • 创建新资源，不指定ID

  • 服务端生成ID并返回

  • 示例：POST /class/students + JSON payload

  PUT - 整体替换资源（幂等）
  • 需要资源ID在路径中

  • 完全替换资源的所有属性

  • 空payload {} 表示重置资源

  • 示例：PUT /class/students/2

  PATCH - 部分更新资源（非幂等）
  • 需要资源ID在路径中

  • 只更新指定的属性，保留其他属性

  • 支持嵌套结构的点分语法更新

  • 数组操作：通过 _arrayop=add|remove 参数控制

  PATCH - 部分更新资源（非幂等）
  • 需要资源ID在路径中

  • 只更新指定的属性，保留其他属性

  • 支持嵌套结构的点分语法更新

  • 数组操作：通过 _arrayop=add|remove 参数控制

  • 示例：PATCH /class/students/2?_arrayop=add

  DELETE - 删除资源（幂等）
  • 支持带payload（虽然HTTP规范模糊，但此规范明确支持）

  • 不支持时可用 DELETE Over POST：POST /resource?_method=DELETE

  • 示例：DELETE /class/students/2