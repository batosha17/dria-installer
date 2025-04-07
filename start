#!/bin/bash
echo "-----------------------------------------------------------------------------"
curl -s https://raw.githubusercontent.com/DOUBLE-TOP/tools/main/doubletop.sh | bash
echo "-----------------------------------------------------------------------------"

echo "Устанавливаем софт (временной диапазон ожидания ~5-20 min.)"
echo "-----------------------------------------------------------------------------"
curl -s https://raw.githubusercontent.com/DOUBLE-TOP/tools/main/main.sh | bash &>/dev/null
curl -s https://raw.githubusercontent.com/DOUBLE-TOP/tools/main/ufw.sh | bash &>/dev/null

# Установка Ollama
echo "Ставим Ollama..."
curl -fsSL https://ollama.com/install.sh | sh &>/dev/null
echo "Установка Ollama завершена"
echo ""

# Установка Dria
echo "Ставим Dria..."
cd $HOME
curl -fsSL https://dria.co/launcher | bash &>/dev/null

# Настройка сервиса
SERVICE_NAME="dria.service"
SERVICE_FILE="/etc/systemd/system/$SERVICE_NAME"
LOG_FILE="/var/log/dria.log"
ERROR_LOG_FILE="/var/log/dria_error.log"
HOME_DIR="$HOME"
WORK_DIR="$HOME_DIR/.dria/bin"
LAUNCHER="dkn-compute-launcher"
ENV_PATH="$HOME_DIR/.dria/dkn-compute-launcher/.env"

# Удаляем предыдущий сервис, если он существует
if systemctl list-units --type=service --all | grep -q "$SERVICE_NAME"; then
    sudo systemctl stop "$SERVICE_NAME"
    sudo systemctl disable "$SERVICE_NAME"
    if [ -f "$SERVICE_FILE" ]; then
        sudo rm "$SERVICE_FILE"
    fi
    > "$ERROR_LOG_FILE"
    sudo systemctl daemon-reload
    echo "Существующий $SERVICE_NAME удален."
fi

# Создание systemd-сервиса
cat <<EOF | sudo tee "$SERVICE_FILE" > /dev/null
[Unit]
Description=Dria Service
After=network.target

[Service]
User=root
WorkingDirectory=$WORK_DIR
ExecStartPre=/bin/bash -c "sed -i 's|^DKN_P2P_LISTEN_ADDR=.*|DKN_P2P_LISTEN_ADDR=/ip4/0.0.0.0/tcp/4002|' ${ENV_PATH}"
ExecStart=${WORK_DIR}/${LAUNCHER} start
Restart=always
RestartSec=5
StandardOutput=append:$LOG_FILE
StandardError=append:$ERROR_LOG_FILE

[Install]
WantedBy=multi-user.target
EOF

# Перезагрузка systemd и запуск сервиса
sudo systemctl daemon-reload
sudo systemctl enable dria.service
sudo systemctl start dria.service

echo "Установка Dria завершена. Статус можна перевірити командою:"
echo "sudo systemctl status dria.service"
