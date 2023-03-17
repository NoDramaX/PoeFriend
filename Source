import time
import cv2
import numpy as np
import mss
import mss.tools
from PyQt5.QtWidgets import QApplication, QWidget, QSlider, QLabel, QVBoxLayout, QCheckBox, QLineEdit, QPushButton, QHBoxLayout, QFrame, QSizePolicy, QSpacerItem
from PyQt5.QtCore import Qt
import threading
import pyautogui
import keyboard
from PyQt5.QtCore import QTimer, QPoint
from PyQt5.QtGui import QCursor

x, y, w, h = 300, 250, 115, 600

templates = [
    cv2.imread('Screenshot_1.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_2.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_3.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_4.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_5.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_6.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_7.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_8.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_9.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_10.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_11.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_12.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_13.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_14.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_15.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_16.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_17.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_18.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_19.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_20.png', cv2.IMREAD_GRAYSCALE),
    cv2.imread('Screenshot_21.png', cv2.IMREAD_GRAYSCALE),
]
w_templates, h_templates = zip(*[template.shape[::-1] for template in templates])

threshold_value = 75
delay_value = value = 00.1
delay_value_float = delay_value / 10.0
captured_position = None
captured_position_2 = None
captured_position_3 = None
captured_position_4 = None


class MainWindow(QWidget):

    def __init__(self, delay_value=0.5):
        self.delay_value = delay_value
        super().__init__()
        self.setWindowTitle("PoeFriend")
        self.setStyleSheet("background-color: rgb(255,255,255);")
        # Создание текстовых полей и кнопки для задания региона поиска
        self.x_input = QLineEdit()
        self.y_input = QLineEdit()
        self.w_input = QLineEdit()
        self.h_input = QLineEdit()
        self.apply_button = QPushButton('Apply Settings')
        self.apply_button.clicked.connect(self.update_region)
        self.apply_button.setFixedSize(125, 25)
        self.apply_button.setStyleSheet("QPushButton {background-color: #F5F5F5; border: 2px solid #A9A9A9; border-radius: 8px;}")
        self.apply_button.setSizePolicy(QSizePolicy.Expanding, QSizePolicy.Fixed)

        self.threshold_slider = QSlider(Qt.Horizontal)
        self.threshold_slider.setMinimum(0)
        self.threshold_slider.setMaximum(100)
        self.threshold_slider.setValue(threshold_value)
        self.threshold_slider.valueChanged.connect(self.update_threshold_label)

        self.threshold_label = QLabel(f"Sample Depth: {threshold_value}")

        self.delay_slider = QSlider(Qt.Horizontal)
        self.delay_slider.setMinimum(0)
        self.delay_slider.setMaximum(1000)
        self.delay_slider.setValue(int(delay_value_float * 100))
        self.delay_slider.valueChanged.connect(self.update_delay_label)

        self.delay_label = QLabel(f"Delay: {delay_value}")

        self.template_checkboxes_left = []
        self.template_checkboxes_right = []
        for i, template in enumerate(templates):
            new_names = ["1. Fusing", "2. Alteration", "3. Chisel", "4. Chromatic", "5. Stacked Deck", "6. Regret",
                         "7. Chaos", "8. Alchemy", "9. Vaal", "10. Scouring", "11. Ancient", "12. Blessed", "13. Regal",
                         "14. ExShard", "15. ~5000 Markers"
                , "16. Augment", "17. Divine Orb", "18. Transmutation", "19. Jeweller Orb", "20. Sextant",
                         "21. GemCutter Prism"]
            checkbox = QCheckBox(new_names[i])
            checkbox.setChecked(True)
            if i < len(templates) // 2:
                self.template_checkboxes_left.append(checkbox)
            else:
                self.template_checkboxes_right.append(checkbox)

        template_box_left = QVBoxLayout()
        template_box_right = QVBoxLayout()
        for checkbox in self.template_checkboxes_left:
            template_box_left.addWidget(checkbox)
        for checkbox in self.template_checkboxes_right:
            template_box_right.addWidget(checkbox)
            self.template_checkboxes = self.template_checkboxes_left.copy()
            self.template_checkboxes.extend(self.template_checkboxes_right)

        template_box = QHBoxLayout()
        template_box.addLayout(template_box_left)
        template_box.addLayout(template_box_right)

        layout = QVBoxLayout()
        layout.addWidget(self.threshold_slider)
        layout.addWidget(self.threshold_label)
        layout.addWidget(self.delay_slider)
        layout.addWidget(self.delay_label)
        layout.addLayout(template_box)

        region_label = QLabel('Region:')
        region_layout = QHBoxLayout()
        region_layout.addWidget(region_label)
        region_layout.addWidget(QLabel('X:'))
        region_layout.addWidget(self.x_input)
        region_layout.addWidget(QLabel('Y:'))
        region_layout.addWidget(self.y_input)
        region_layout.addWidget(QLabel('Width:'))
        region_layout.addWidget(self.w_input)
        region_layout.addWidget(QLabel('Height:'))
        region_layout.addWidget(self.h_input)

        spacer = QSpacerItem(1, 1, QSizePolicy.Expanding, QSizePolicy.Minimum)
        layout.addLayout(region_layout)
        layout.addItem(spacer)
        layout.addWidget(self.apply_button, 0, Qt.AlignCenter)
        layout.addItem(spacer)

        capture_layout_1 = QHBoxLayout()
        self.capture_button = QPushButton('Захватить позицию 1')
        self.capture_button.setToolTip(
            "<font size=5>This button is responsible for capturing the coordinates of the position where the store update is located.</font>")
        self.capture_button.clicked.connect(self.capture_position)
        self.capture_button.setFixedSize(150, 25)
        self.capture_button.setStyleSheet("QPushButton {background-color: #F5F5F5; border: 2px solid #A9A9A9; border-radius: 8px;}")
        capture_layout_1.addWidget(self.capture_button)
        self.capture_label_1 = QLabel('Position 1: Not captured')
        capture_layout_1.addWidget(self.capture_label_1)
        layout.addLayout(capture_layout_1)

        capture_layout_2 = QHBoxLayout()
        self.capture_button_2 = QPushButton('Захватить позицию 2')
        self.capture_button_2.setToolTip(
            "<font size=5>This button is responsible for capturing the coordinates of the position where the slider's initial position cursor should be..</font>")
        self.capture_button_2.clicked.connect(self.capture_position_2)
        self.capture_button_2.setFixedSize(150, 25)
        self.capture_button_2.setStyleSheet("QPushButton {background-color: #F5F5F5; border: 2px solid #A9A9A9; border-radius: 8px;}")
        capture_layout_2.addWidget(self.capture_button_2)
        self.capture_label_2 = QLabel('Position 2: Not captured')
        capture_layout_2.addWidget(self.capture_label_2)
        layout.addLayout(capture_layout_2)

        capture_layout_3 = QHBoxLayout()
        self.capture_button_3 = QPushButton('Захватить позицию 3')
        self.capture_button_3.setToolTip(
            "<font size=5>This button is responsible for capturing the coordinates of the position where the cursor of the end position of the slider should be..</font>")
        self.capture_button_3.clicked.connect(self.capture_position_3)
        self.capture_button_3.setFixedSize(150, 25)
        self.capture_button_3.setStyleSheet("QPushButton {background-color: #F5F5F5; border: 2px solid #A9A9A9; border-radius: 8px;}")
        self.capture_label_3 = QLabel('Position 3: Not captured')
        capture_layout_3.addWidget(self.capture_button_3)
        capture_layout_3.addWidget(self.capture_label_3)
        layout.addLayout(capture_layout_3)

        capture_layout_4 = QHBoxLayout()
        self.capture_button_4 = QPushButton('Захватить позицию 4')
        self.capture_button_4.setToolTip(
            "<font size=5>This button is responsible for capturing the coordinates of the position where the accept trade button is located.</font>")
        self.capture_button_4.clicked.connect(self.capture_position_4)
        self.capture_button_4.setFixedSize(150, 25)
        self.capture_button_4.setStyleSheet("QPushButton {background-color: #F5F5F5; border: 2px solid #A9A9A9; border-radius: 8px;}")
        capture_layout_4.addWidget(self.capture_button_4)
        self.capture_label_4 = QLabel('Position 4: Not captured')
        capture_layout_4.addWidget(self.capture_label_4)
        layout.addLayout(capture_layout_4)

        version_layout = QVBoxLayout()
        version_label = QLabel('Beta 1.0')
        version_label.setAlignment(Qt.AlignCenter)
        version_label.setStyleSheet('font-size: 8pt')
        version_layout.addWidget(version_label)
        layout.addLayout(version_layout)

        version_layout = QVBoxLayout()
        version_label = QLabel('Developer: https://github.com/NoDramaX')
        version_label.setAlignment(Qt.AlignCenter)
        version_label.setStyleSheet('font-size: 8pt')
        version_layout.addWidget(version_label)
        layout.addLayout(version_layout)

        version_layout = QVBoxLayout()
        version_label = QLabel('https://discord.gg/JP8JDkuUbp')
        version_label.setAlignment(Qt.AlignCenter)
        version_label.setStyleSheet('font-size: 8pt')
        version_layout.addWidget(version_label)
        layout.addLayout(version_layout)
        self.resize(400, 500)

        self.setLayout(layout)

    def update_threshold_label(self, value):
        self.threshold_label.setText(f"Sample Depth: {value}")
        global threshold_value
        threshold_value = value

    def update_delay_label(self, value):
        float_value = value / 1000.0
        self.delay_value = float_value
        self.delay_slider.setValue(value)
        self.delay_label.setText(f"Delay: {self.delay_value:.3f}")

    def capture_position(self):
        self.capture_button.setEnabled(True)
        self.capture_button.setText('Захват...')
        QTimer.singleShot(2000, self.capture_cursor_position)

    def capture_cursor_position(self):
        global captured_position
        cursor_position = QCursor.pos()
        captured_position = (cursor_position.x(), cursor_position.y())
        self.capture_button.setText('Позиция захвачена!')
        self.capture_button.setEnabled(True)
        self.capture_button.setText('Захватить другую позицию')
        self.update_position_label_1()
        print(captured_position)

    def update_position_label_1(self):
        global captured_position
        self.capture_label_1.setText(f"Position 1: {captured_position}")

    def capture_position_2(self):
        self.capture_button_2.setEnabled(True)
        self.capture_button_2.setText('Захват...')
        QTimer.singleShot(2000, self.capture_cursor_position_2)

    def capture_cursor_position_2(self):
        global captured_position_2
        cursor_position = QCursor.pos()
        captured_position_2 = (cursor_position.x(), cursor_position.y())
        self.capture_button_2.setText('Позиция захвачена!')
        self.capture_button_2.setEnabled(True)
        self.capture_button_2.setText('Захватить другую позицию')
        self.update_position_label_2()
        print(captured_position_2)

    def update_position_label_2(self):
        global captured_position_2
        self.capture_label_2.setText(f"Position 2: {captured_position_2}")

    def capture_position_3(self):
        self.capture_button_3.setEnabled(True)
        self.capture_button_3.setText('Захват...')
        QTimer.singleShot(2000, self.capture_cursor_position_3)

    def capture_cursor_position_3(self):
        global captured_position_3
        cursor_position = QCursor.pos()
        captured_position_3 = (cursor_position.x(), cursor_position.y())
        self.capture_button_3.setText('Позиция захвачена!')
        self.capture_button_3.setEnabled(True)
        self.capture_button_3.setText('Захватить другую позицию')
        self.update_position_label_3()
        print(captured_position_3)

    def update_position_label_3(self):
        global captured_position_3
        self.capture_label_3.setText(f"Position 3: {captured_position_3}")

    def capture_position_4(self):
        self.capture_button_4.setEnabled(True)
        self.capture_button_4.setText('Захват...')
        QTimer.singleShot(2000, self.capture_cursor_position_4)

    def capture_cursor_position_4(self):
        global captured_position_4
        cursor_position = QCursor.pos()
        captured_position_4 = (cursor_position.x(), cursor_position.y())
        self.capture_button_4.setText('Позиция захвачена!')
        self.capture_button_4.setEnabled(True)
        self.capture_button_4.setText('Захватить другую позицию')
        self.update_position_label_4()
        print(captured_position_4)

    def update_position_label_4(self):
        global captured_position_4
        self.capture_label_4.setText(f"Position 4: {captured_position_4}")
   
    def update_region(self):
        global x, y, w, h
        try:
            x = int(self.x_input.text())
            y = int(self.y_input.text())
            w = int(self.w_input.text())
            h = int(self.h_input.text())
        except ValueError:
            pass


    exit_flag = False
def search_templates():
    with mss.mss() as sct:
        while True:
            found_template = False
            monitor = {"top": y, "left": x, "width": w, "height": h}
            screen = np.array(sct.grab(monitor))
            
            gray_screen = cv2.cvtColor(screen, cv2.COLOR_BGR2GRAY)

            threads = [] 

            for i, (template, w_template, h_template) in enumerate(zip(templates, w_templates, h_templates)):
                if not main_window.template_checkboxes[i].isChecked():
                    continue
                res = cv2.matchTemplate(gray_screen, template, cv2.TM_CCOEFF_NORMED)
                loc = np.where(res >= threshold_value/100)

                if len(loc[0]) > 0:
                    found_template = True
                    for pt in zip(*loc[::-1]):
                        cv2.rectangle(screen, pt, (pt[0] + w_template, pt[1] + h_template), (0, 255, 0), 2)

                        thread = threading.Thread(target=process_template_matches,
                                                  args=(pt, w_template, h_template, x, y))
                        threads.append(thread)  

                        thread.start()

            for thread in threads:
                thread.join()

            cv2.imshow('Screen', screen)
            cv2.waitKey(1) 
            if keyboard.is_pressed('caps lock'):

                if not found_template:
                    pyautogui.moveTo(captured_position)
                    print("Неть")
                    pyautogui.click()
                    time.sleep(0.7)

            if cv2.waitKey(1) == ord('q'):
                cv2.destroyAllWindows()
                break

def process_template_matches(pt, w_template, h_template, x, y):
    if keyboard.is_pressed('caps lock'):
        pyautogui.moveTo(pt[0] + x + w_template / 2, pt[1] + y + h_template / 2)
        time.sleep(main_window.delay_value)
        pyautogui.click()
        time.sleep(main_window.delay_value)
        pyautogui.moveTo(captured_position_2)
        pyautogui.mouseDown(button='left')
        pyautogui.moveTo(captured_position_3)
        time.sleep(main_window.delay_value)
        pyautogui.mouseUp(button='left')
        pyautogui.moveTo(captured_position_4)
        pyautogui.click()
        time.sleep(main_window.delay_value)
        pyautogui.click()




app = QApplication([])
main_window = MainWindow()
main_window.show()
thread = threading.Thread(target=search_templates)
thread.start()
app.exec_()
