 提示词文件位置： ./GEMINI.md
 # 切换模型
 
 ```bash
 gemini -m "gemini-2.5 flash"
 ```

 # 配置位置
 ~/.gemini/settings.json
 .gemini/settings.json
 /mcp 查看配置

-m 模型
-p prompt （使用prompt运行后立即退出）
-d 启动调试
-yolo 自动批准所有工具调用
# 运行时参数
/chat save 保存会话
/chat share 会话导出为markdown/json
/clear 清除会话上下文
/compress 压缩上下文
/copy  将最近结果复制到剪贴板
/auth 修改认证方法
/init 当前目录下创建GEMINI.md模版文件