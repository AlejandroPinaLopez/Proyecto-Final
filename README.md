# Proyecto-Final
"Codigo de trackeo de un objeto para interpretar su movimiento como el tecleo de alguna tecla, movimiento de raton etc."

#include <opencv2/core.hpp>
#include <opencv2/videoio.hpp>
#include <opencv2/highgui.hpp>
#include <iostream>
#include <opencv2/opencv.hpp>
#include <vector>
#include <Windows.h>
#include <tchar.h>


using namespace cv;
using namespace std;

//Funtions to simulate key presing
void PressKey(BYTE key, HWND hwndGame) {
	if (hwndGame != NULL) {
		SetForegroundWindow(hwndGame);
		keybd_event(key, 0, 0, 0); // Key down
		Sleep(100);//Time between clicks
		keybd_event(key, 0, KEYEVENTF_KEYUP, 0); // Key up
	}
	else {
		cout << "No se encontró la ventana del juego." << endl;
	}
}
void PressKeys(BYTE key1, BYTE key2, HWND hwndGame) {
	if (hwndGame != NULL) {
		SetForegroundWindow(hwndGame);
		keybd_event(key1, 0, 0, 0); // Key1 down
		keybd_event(key2, 0, 0, 0); // Key2 down
		Sleep(150); //Time between clicks
		keybd_event(key1, 0, KEYEVENTF_KEYUP, 0); // Key1 up
		keybd_event(key2, 0, KEYEVENTF_KEYUP, 0); // Key2 up
	}
	else {
		cout << "No se encontró la ventana del emulador." << endl;
	}
}

int main(){

	//Mat image = imread("dragon ball super by dt501061 on DeviantArt.jpg");
	//Mat image_gray;
	//Mat image_hsv;
	Mat frame, image_hsv, mask;
	VideoCapture cap(0);
	double maxArea;
	vector<Point> largestContour;
	int lowerHue = 87, lowerSaturation = 1, lowerValue = 116;
	int upperHue = 255, upperSaturation = 255, upperValue = 255;

	if (!cap.isOpened()) {
	cout << "Error al leer la camara" << endl;
	return 1;
	}

	namedWindow("Settings", WINDOW_AUTOSIZE);

	HWND hwndGame = FindWindow(NULL, _T("Vampire Survivors"));

	/*
	createTrackbar("Lower Hue", "Settings", &lowerHue, 255, nullptr);
	createTrackbar("Upper Hue", "Settings", &upperHue, 255, nullptr);
	createTrackbar("Lower Saturation", "Settings", &lowerSaturation, 255, nullptr);
	createTrackbar("Upper Saturation", "Settings", &upperSaturation, 255, nullptr);
	createTrackbar("Lower Value", "Settings", &lowerValue, 255, nullptr);
	createTrackbar("Upper Value", "Settings", &upperValue, 255, nullptr);*/
	
	
	//Almacenamiento de frames
	while (1) {

		cap >> frame;

		if (frame.empty())
			break;

		int rectWidth = frame.cols / 3 * 1;
		int rectHeight = frame.rows / 3 * 1;

		//Creation of rectangules
		Rect rightRect(0, frame.rows / 3 + frame.rows / 6 - rectHeight / 2, rectWidth, rectHeight);
		Rect leftRect(frame.cols - rectWidth, frame.rows / 3 + frame.rows / 6 - rectHeight / 2, rectWidth, rectHeight);
		Rect upRect(frame.cols / 3 + frame.cols / 6 - rectWidth / 2, 0, rectWidth, rectHeight);
		Rect downRect(frame.cols / 3 + frame.cols / 6 - rectWidth / 2, frame.rows - rectHeight, rectWidth, rectHeight);

		Rect topLeftRect(0, 0, rectWidth, rectHeight);
		Rect topRightRect(frame.cols - rectWidth, 0, rectWidth, rectHeight);
		Rect bottomLeftRect(0, frame.rows - rectHeight, rectWidth, rectHeight);
		Rect bottomRightRect(frame.cols - rectWidth, frame.rows - rectHeight, rectWidth, rectHeight);

		Rect escRect(0, 0, rectWidth, rectHeight);
		Rect enterRect(frame.cols - rectWidth, 0, rectWidth, rectHeight);

		cvtColor(frame, image_hsv, COLOR_BGR2HSV);

		Scalar lowerTrehold(lowerHue, lowerSaturation, lowerValue);
		Scalar upperTrehold(upperHue, upperSaturation, upperValue);

		inRange(image_hsv, lowerTrehold, upperTrehold, mask);
		vector<vector<Point>> contours;
		findContours(mask, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);

		//Biggest contour preference
		maxArea = 0.0;
		for (int i = 0; i < contours.size(); i++) {
			double area = contourArea(contours[i]);
			if (area > maxArea) {
				maxArea = area;
				largestContour = contours[i];
			}
		}

		if (!largestContour.empty()) {
			//Get the center of the object
			Moments m = moments(largestContour);
			int cx = int(m.m10 / m.m00);
			int cy = int(m.m01 / m.m00);

			vector<vector<Point>> contoursToDraw = { largestContour };
			drawContours(frame, contoursToDraw, -1, (0, 255, 0), 3);
			circle(frame, Point(cx, cy), 8, Scalar(255, 255, 255), 1);


			//Drawing rectangules
			rectangle(frame, leftRect, Scalar(0, 0, 255), 2);
			rectangle(frame, rightRect, Scalar(0, 0, 255), 2);
			rectangle(frame, upRect, Scalar(0, 0, 255), 2);
			rectangle(frame, downRect, Scalar(0, 0, 255), 2);

			rectangle(frame, topRightRect, Scalar(0, 255, 0), 2);
			rectangle(frame, topLeftRect, Scalar(0, 255, 0), 2);
			rectangle(frame, bottomRightRect, Scalar(0, 255, 0), 2);
			rectangle(frame, bottomLeftRect, Scalar(0, 255, 0), 2);

			//Interpretation the center of the object inside the rectangules as keyboard keys
			
			if (leftRect.contains(Point(cx, cy))) {
				cout << "LEFT" << endl;
				PressKey(VK_LEFT, hwndGame);
			}
			else if (rightRect.contains(Point(cx, cy))) {
				cout << "RIGHT" << endl;
				PressKey(VK_RIGHT, hwndGame);
			}
			else if (upRect.contains(Point(cx, cy))) {
				cout << "UP" << endl;
				PressKey(VK_UP, hwndGame);
			}
			else if (downRect.contains(Point(cx, cy))) {
				cout << "DOWN" << endl;
				PressKey(VK_DOWN, hwndGame);
			}
			else if (topRightRect.contains(Point(cx, cy))) { //Press two different keys at the same time
				cout << "UP-LEFT" << endl;
				PressKeys(VK_LEFT, VK_UP, hwndGame); 
			}
			else if (topLeftRect.contains(Point(cx, cy))) {
				cout << "UP-RIGHT" << endl;
				PressKeys(VK_RIGHT, VK_UP, hwndGame);
			}
			else if (bottomRightRect.contains(Point(cx, cy))) {
				cout << "DOWN-LEFT" << endl;
				PressKeys(VK_LEFT, VK_DOWN, hwndGame);
			}
			else if (bottomLeftRect.contains(Point(cx, cy))) {
				cout << "DOWN-RIGHT" << endl;
				PressKeys(VK_RIGHT, VK_DOWN, hwndGame);
			}
			

		}

		imshow("Settings", mask);
		imshow("Original frame", frame);


		char c = (char)waitKey(25);
		if (c == 27)
			break;		

	}

	cap.release();
	destroyAllWindows();

	return 0;
}
