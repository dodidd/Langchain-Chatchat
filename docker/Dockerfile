# 基础镜像
FROM python:3.11
LABEL maintainer=Langchain-Chatchat
WORKDIR /root

# 环境变量
ENV CHATCHAT_ROOT=/root/chatchat_data

# 初始化环境
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone
# 2. 安装系统依赖（清理缓存减少镜像体积）
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    git \
    libgl1 \
    libglib2.0-0 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 3. 配置 pip 国内镜像源并升级工具
RUN pip install --upgrade pip setuptools wheel -i https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip install pipx -i https://pypi.tuna.tsinghua.edu.cn/simple

# 4. 用 pipx 安装指定版本的 Poetry（显式传递镜像源）
RUN pipx install --pip-args="--index-url https://pypi.tuna.tsinghua.edu.cn/simple" poetry==1.7.0
ENV PATH="/root/.local/bin:${PATH}"

# 5. 克隆项目（浅克隆加快速度）
RUN git clone --depth 1 https://github.com/chatchat-space/Langchain-Chatchat.git

# 安装依赖
WORKDIR /root/Langchain-Chatchat/libs/chatchat-server
RUN poetry config virtualenvs.create false
RUN poetry install --with lint,test -E xinference

## 确保 Python 可以找到 chatchat 模块
ENV PYTHONPATH="/root/Langchain-Chatchat/libs/chatchat-server:${PYTHONPATH}"

# 初始化配置
WORKDIR /root/Langchain-Chatchat/libs/chatchat-server/chatchat
RUN python cli.py init

# 初始化知识库文件
ADD data.tar.gz $CHATCHAT_ROOT/

EXPOSE 7861 8501
ENTRYPOINT ["python", "cli.py", "start", "-a"]
