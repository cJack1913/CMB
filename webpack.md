1. 安装[Node.js](https://nodejs.org/en/download/)，Node.js自带npm，npm是一个JavaScript的包管理工具
2. 安装cnpm
	
		npm install -g cnpm --registry=https://registry.npm.taobao.org 
	npm的服务器在国外，国内通常使用cnpm（淘宝镜像），公司默认开通了npm.taobao.org的访问权限（关闭安全桌面的状态下）
3. 安装Webpack和Webpack开发工具
		
		cd 项目目录
		# 全局安装
		cnpm install webpack -g
		# 再项目安装
		cnpm install webpack --save-dev
		# 全局安装
		cnpm install webpack-dev-server -g
		# 开发工具
		cnpm install webpack-dev-server --save-dev
