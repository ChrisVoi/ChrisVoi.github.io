---
title: "C++ Signalgenerator"
author: "Christopher Voigtmann"
date: "2020-09-18"
meta_img: images/qtrect.JPG
---

![qtrect](/images/qtrect.JPG)


Mit der Programmierumgebung QT habe ich einen Rechtecksignal-Generator erstellt. Wie dieser im Detail funktioniert, ist in dem Post für den Signalgenarator auf Java Basis zu sehen. Bei QT ging es viel mehr um die Erstellung eines User-Interfaces mit Hilfe der vorgegebenen Zeichenmöglichkeiten. Daher ist hier auch nur der Code für das "Mainwindow" zu sehen.




##### Code Mainwindow Header


```cpp

#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include "rectangleData.h"
#include "dataBuffer.h"
#include <QTimer>
#include <QPaintEvent>
#include <QPainter>
#include <QMouseEvent>


QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private slots:
    void on_pushStart_clicked();

    void on_pushStop_clicked();

    void on_pushRestart_clicked();

```
##### Code Mainwindow Main

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    :QMainWindow(parent)
    ,ui(new Ui::MainWindow),minValue(-20),maxValue(20),n(75),noise(5),curN(0)
    ,scaleMax(25),scaleMin(-25),delta(1),xScale(0),mPosX(0),mPosY(0),mWert(0)
    ,run(false),mousInFrame(false)
    ,myRect(minValue,maxValue,n,noise) //Initialisiere Rechteck-Generator
    ,myBuff(600-1)                     //Initialisiere Zwischenspeicher

{

    ui->setupUi(this);



    myTimer = new QTimer(this);
    connect(myTimer, SIGNAL(timeout()),this,SLOT(timeIsUp()));

  //Initialisiere statischen Teil des User-Interfaces
  ui->dial_maxValue->setRange(25,100);
  ui->dial_maxValue->setValue(scaleMax);

  ui->dial_minValue->setRange(25,100);
  ui->dial_minValue->setValue(-scaleMin);

  ui->label_maxValue->setNum(scaleMax);
  ui->label_minValue->setNum(scaleMin);

  ui->Slider_noise->setRange(-5,5);
  ui->Slider_noise->setEnabled(false);

  ui->checkBox_run->setEnabled(false);
  ui->checkBox_run->setChecked(true);

  ui->lcd_maxValue->display(scaleMax);
  ui->lcd_minValue->display(scaleMin);

  ui->Scrollbar_UseData->setRange(0,ui->frame->width()-1);
  ui->Scrollbar_UseData->setValue(0);
  ui->Scrollbar_UseData->setPageStep(5);
}


MainWindow::~MainWindow()
{
    delete ui;
}


void MainWindow::on_pushStart_clicked()
{
   run = true;
   ui->checkBox_run->setChecked(false);
   myTimer->start(20);
}


void MainWindow::on_pushStop_clicked()
{
   myTimer->stop();
   run = false;
   ui->checkBox_run->setChecked(true);

}


void MainWindow::on_pushRestart_clicked()
{
    //Lösche Inhalte Daten-Array und Zwischenspeicher
    for (int i = 0; i < ui->frame->width() ; i++)
    {
        myBuff.input(0);
        vX[i]=0;
        vY[i]=0;
    }

    run = true;

    myTimer->start(20);

    ui->checkBox_run->setChecked(false);
}

void MainWindow::paintEvent(QPaintEvent *event)
{
  QPainter myPainter(this);

  event->ignore();

  //Zeichnen der Mittellinie (Null-Linie)
  myPainter.setPen(Qt::green);
  myPainter.drawLine(ui->frame->x(),framestart+(ui->frame->height()/(relation+1))*relation,ui->frame->x()+ui->frame->width(),framestart+(ui->frame->height()/(relation+1))*relation);


   //Zeichne beschriftetes Array
   myPainter.setPen(Qt::black);
   for (int i =1 ; i < ui->frame->width() ; i++)
    {
       myPainter.drawLine(vX[i],vY[i],vX[i-1],vY[i-1]);
    }

    //Focus & Context, Zeichne Positions-Zahlenwert, wenn Mausposition in Frame
    if(mousInFrame==true)
    {
       myPainter.drawText(mPosX,mPosY, QString::number(-(mWert-(framestart + ui->frame->height()/(relation+1)*relation))*hole/ui->frame->height()));
    }
}


void MainWindow::timeIsUp()
{
    //Füllen des Zwischenspeichers mit einem Datenwert pro Zyklus
    myBuff.input(myRect.getData());

    //Beschreibe temporäre und globale Daten
    double tmprfw =ui->frame->width();

    delta = double((tmprfw/(tmprfw-xScale+1)));

    if (delta<1)
    {
        delta =1;
    }

    relation = scaleMax/-(scaleMin);
    hole = scaleMax - scaleMin;
    framestart= ui->frame->y()+26;


    //Aktualisiere dynamische Beschriftung Y-Achse
    ui->label_min20->move(ui->frame->x()-40,framestart -35 + ui->frame->height()/(relation+1)*relation + 20*ui->frame->height()/hole);
    ui->line_min20->move(ui->frame->x()-20,framestart -35 + ui->frame->height()/(relation+1)*relation + 20*ui->frame->height()/hole);

    ui->label_zero->move(ui->frame->x()-30,framestart-35+(ui->frame->height()/(relation+1))*relation);
    ui->line_zero->move(ui->frame->x()-20,framestart-35+(ui->frame->height()/(relation+1))*relation);

    ui->label_max20->move(ui->frame->x()-40,framestart -35 + ui->frame->height()/(relation+1)*relation + (-20)*ui->frame->height()/hole);
    ui->line_max20->move(ui->frame->x()-20,framestart -35 + ui->frame->height()/(relation+1)*relation + (-20)*ui->frame->height()/hole);

    //Beschreibe Array
    for (int i = 0; i < ui->frame->width();i++ )
    {
        vY[i] =framestart + ui->frame->height()/(relation+1)*relation + myBuff.getValue(i)*ui->frame->height()/hole;

        vX[i] = ui->frame->x() + ui->frame->width()  -i*(delta);

        //Halte Frame-Grenzen ein
        if (vX[i]<ui->frame->x())
        {
             vX[i] =ui->frame->x();
        }

        if (vX[i]>ui->frame->x()+ui->frame->width())
        {
             vX[i] =ui->frame->x();
        }

        if (vY[i]<framestart)
        {
             vY[i] =framestart;
        }

        if (vY[i]>framestart+ui->frame->height())
        {
             vY[i] =framestart+ui->frame->height();
        }
    }


    //Focus & Context, Verarbeitung der übergebenen Mausposition
    if (mPosX<ui->frame->width()+ui->frame->x()&&mPosX>ui->frame->x())
    {
    if (mPosY<framestart+ui->frame->height()&& mPosY>framestart)
    {
        mousInFrame = true;

        for (int i =ui->frame->width(); i > 0; i--)
        {
            if (mPosX<ui->frame->x()+ui->frame->width()-i&&mPosX>ui->frame->x()+ui->frame->width()-i*delta)
            {
                mWert = vY[i];
            }

            else if (mPosX == vX[i] && delta == 1)
            {
                 mWert = vY[i];
            }

        }

    }
    else
         mousInFrame = false;
    }
    else
         mousInFrame = false;


    //Beschrifte LCD-Anzeigen
    double actNr = -(vY[0]-(framestart + ui->frame->height()/(relation+1)*relation))*hole/ui->frame->height();
    double actNrchill =0 ;
    double noise;

    if (actNr > 0)
    {
        actNrchill = 20; //direktes Abgreifen der Daten mit get()-Methoden im Rechteckgenerator möglich
    }

    else if (actNr < 0)
    {
        actNrchill = -20; //direktes Abgreifen der Daten mit get()-Methoden im Rechteckgenerator möglich
    }

    noise = -actNr+actNrchill;

    ui->lcdActNum->display(actNr);
    ui->lcdActNum_chill->display(actNrchill);

    ui->Slider_noise->setValue(noise);
    curN=ui->frame->width()-xScale;
    ui->lcdUsedData->display(curN);
    ui->lcd_delta->display(delta);

    //Locke aktuellen Wert der Einstellungsmöglichkeiten
    ui->Scrollbar_UseData->setValue(xScale);

    ui->dial_maxValue->setValue(scaleMax);
    ui->dial_minValue->setValue(-scaleMin);

    repaint();
}


void MainWindow::mouseMoveEvent(QMouseEvent *e)

{
    mPosX = e->x();
    mPosY = e->y();
}


void MainWindow::on_dial_maxValue_valueChanged(int value)
{
    if (run == false)
    {
        scaleMax=value;
        ui->lcd_maxValue->display(scaleMax);
        ui->label_maxValue->setNum(scaleMax);
    }
}


void MainWindow::on_dial_minValue_valueChanged(int value)
{
    if (run == false)
    {
        scaleMin = -value;
        ui->lcd_minValue->display(scaleMin);
        ui->label_minValue->setNum(scaleMin);
    }

}


void MainWindow::on_Scrollbar_UseData_sliderMoved(int position)
{
    if (run == false)
    {
        xScale = position;
        curN=ui->frame->width()-xScale;
        ui->lcdUsedData->display(curN);
    }
}


void MainWindow::on_pushQuit_clicked()
{
     QApplication::exit();
}


```






