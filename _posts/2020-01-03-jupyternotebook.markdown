# 1 在虚拟环境中使用 Jupyter notebook

- 在虚拟环境中安装ipykernel：pip install ipykernel
- 查看当前 jupyter 中有哪些 kernel: jupyter kernelspec list
- 删除指定 kernel: jupyter kernelspec remove kernel_name
- 进入虚拟环境添加kernel: python -m ipykernel install --user --name 虚拟环境名 --display-name 要显示的名字
- 执行进入： jupyter notebook