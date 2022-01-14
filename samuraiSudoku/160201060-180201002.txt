# -*- coding: utf-8 -*-
from threading import Thread
import time
import tkinter as tk
import random
from tkinter import filedialog
from matplotlib.figure import Figure
from matplotlib.backends.backend_tkagg import (FigureCanvasTkAgg, 
NavigationToolbar2Tk)


#Global Variables
windowX = 1000
windowY = 700
SudokuXMargin = 100
SudokuYMargin = 50
SudokuCellSize = 25
leftBackground = "#e6e6e6" 
rightBackground = "#131926"
start_time = None
allSteps = []
trueSteps = []

def removeFromTrueSteps(element):
    global trueSteps
    trueSteps.remove([element])
    
def addToTrueSteps(element):
    global trueSteps
    trueSteps.append([element])

def addToAllSteps(element):
    global allSteps
    allSteps.append([element])

class cell:
    """
     Bu sınıf, sudoku ızgarasındaki her kareyi temsil eder.
    """
    def __init__(self,value : int,  realValue):
        """
         Parametreler:
             Value  - hücre değeri.
             realValue - gerçek değer, ilk başta verilen karedeki sayıdır.
        """
        self.cellValue = value
        self.realValue = realValue
    
        
    def __str__(self):
        return str(self.cellValue)
    
    def __repr__(self):
        return str(self.cellValue)

class sudokuSolver:
    def __init__(self,board,isSucceded,threadName):
        self.board = board
        self.threadName = threadName
        
        if(self.solve_sudoku(self.board) != False):
            isSucceded[0] = True
        else:
            isSucceded[0] = False
        
    def find_empty_location(self,arr, l):
        for row in range(9):
            for col in range(9):
                if( arr[row][col].cellValue == 0):
                    l[0]= row
                    l[1]= col
                    return True
        return False
    def used_in_row(self, arr, row, num):
        for i in range(9):
            if(arr[row][i].cellValue == num):
                return True
        return False
    
    def used_in_col(self, arr, col, num):
        for i in range(9):
            if(arr[i][col].cellValue == num):
                return True
        return False
    
    def used_in_box(self, arr, row, col, num):
        for i in range(3):
            for j in range(3):
                if(arr[i + row][j + col].cellValue == num):
                    return True
        return False
    
    def check_location_is_safe(self,arr, row, col, num):
        return (not self.used_in_row(arr, row, num)
                and not self.used_in_col(arr, col, num)
                and not self.used_in_box(arr, row - row % 3, col - col % 3, num))
    
    def solve_sudoku(self,arr):
        global trueSteps,allSteps  
        l =[0, 0]
         
        if(not self.find_empty_location(arr, l)):
            return True
        
        row = l[0]
        col = l[1]
        
        for num in range(1, 10):
            record = str("Thread Name:"+ self.threadName+ " row:"+str(row)+" col:"+str(col)+" Number:"+str(num)+" time:"+str(time.time()-start_time))

            
            if(self.check_location_is_safe(arr,
                              row, col, num)):
                 
                arr[row][col].cellValue = num
                
                if(self.solve_sudoku(arr)):
                    Thread(target=addToTrueSteps,args=(record,)).start()
                    return True
     
                arr[row][col].cellValue = 0
                 
        return False
    
    

class sudokuSolverReverse(sudokuSolver):
    """
    normal sudoku çözer gibi.
     ama boş hücreyi sağdan sola ve aşağıdan yukarı doğru kontrol ettiren kısım.
    """
    def __init__(self,board,isSucceded):
        self.board = board
        self.stop = False
        
        if(self.solve_sudoku(self.board) != False):
            isSucceded[0] = True
        else:
            isSucceded[0] = False

    def find_empty_location(self,arr, l):
        for row in range(8,-1,-1):
            for col in range(8,-1,-1):
                if( arr[row][col].cellValue == 0):
                    l[0]= row
                    l[1]= col
                    return True
                
    def solve_sudoku(self,arr):
        l =[0, 0]
        if(self.stop):
            return False
         
        if(not self.find_empty_location(arr, l)):
            return True
        
        row = l[0]
        col = l[1]
        
        for num in range(1, 10):
            
            if(self.check_location_is_safe(arr,
                              row, col, num)):
                 
                arr[row][col].cellValue = num
                
                if(self.solve_sudoku(arr)):
                    return True
     
                arr[row][col].cellValue = 0
        self.resetIfStuck()
        return False
    
    def resetIfStuck(self):
        self.stop = True
        
        

class centerSudoku:
    """
     merkez sudoku sınıfı. 
    Tüm sudoku çözme yolları merkez sudokunun çözümüne göre türer. Bu kısımda bunu gerçekleştiriyoruz.
    """
    def __init__(self,board,isFinished,allowCenter,centerNext,steps,allSteps,threadName):
        
        self.__board = board
        self.allowCenter = allowCenter
        self.centerNext = centerNext
        self.threadName = threadName
        
        self.isFinished = isFinished
        self.result = ""
        self.steps = steps
        self.allSteps = allSteps
        
        a = Thread(target=self.solve_sudoku,args=(board,))
        a.start()
        
        
        self.waitUntilResultShows()
        
        
            
        
    def waitUntilResultShows(self):
        self.isFinished[0] = None
        while True:
            if(self.result not in [True,False]):
                time.sleep(0.01)
            else:
                break
            
        
        if(self.result == True):
            self.isFinished[0] = True
        elif(self.result == False):
            self.isFinished[0] = False
            
            
    def is_used_left_upper(self, arr, row, column, num):
        """
         bu fonksiyon, iki sudokuda uyması gereken hücreleri kontrol eder.
        """
        if(row in [6,7,8] and column in [6,7,8]):
            None
        else:
            return False
        for i in range(15):
            if(arr[row][i].cellValue == num):
                return True
    
        for i in range(15):
            if(arr[i][column].cellValue == num):
                return True
    
                
        return False
    
    def is_used_right_upper(self, arr, row, column, num):
        """
         bu fonksiyon, iki sudokuda uyması gereken hücreleri kontrol eder.
        """
        if(row in [6 ,7 ,8] and column in [12,13,14]):
            None
        else:
            return False
        
        for i in range(6,21):
            if(arr[row][i].cellValue == num):
                return True
    
        for i in range(15):
            if(arr[i][column].cellValue == num):
                return True
    
        return False


    def is_used_right_bottom(self, arr, row, column, num):
        """
         bu fonksiyon, iki sudokuda uyması gereken hücreleri kontrol eder.
        """
        if(row in [12,13,14] and column in [12,13,14]):
            None
        else:
            return False
        for i in range(6,21):
            if(arr[row][i].cellValue == num):
                return True
    
        for i in range(6,21):
            if(arr[i][column].cellValue == num):
                return True
    
        return False
    
    def is_used_left_bottom(self, arr, row, column, num):
        """
        bu fonksiyon, iki sudokuda uyması gereken hücreleri kontrol eder.
        """
        
        if(row in [12,13,14] and column in [6,7,8]):
            None
        else:
            return False
        for i in range(15):
            if(arr[row][i].cellValue == num):
                return True
    
        for i in range(6,21):
            if(arr[i][column].cellValue == num):
                return True
    
        return False
    
    def used_in_row(self, arr, row, num):
        for i in range(6,15):
            if(arr[row][i].cellValue == num):
                return True
        return False
    
    def used_in_col(self, arr, col, num):
        for i in range(6,15):
            if(arr[i][col].cellValue == num):
                return True
        return False
    
    def used_in_box(self, arr, row, col, num):
        for i in range(3):
            for j in range(3):
                if(arr[i + row][j + col].cellValue == num):
                    return True
        return False
    
    def check_location_is_safe(self,arr, row, col, num):
        return (not self.used_in_row(arr, row, num)
                and not self.used_in_col(arr, col, num)
                and not self.used_in_box(arr, row - row % 3, col - col % 3, num)
                and not self.is_used_left_upper(arr,row,col,num)
                and not self.is_used_left_bottom(arr,row,col,num)
                and not self.is_used_right_bottom(arr,row,col,num)
                and not self.is_used_right_upper(arr,row,col,num))

    def find_empty_location(self,arr, l):
        for row in range(6,15):
            for col in range(6,15):
                if( arr[row][col].cellValue == 0):
                    l[0]= row
                    l[1]= col
                    return True
        return False
    
    def solve_sudoku(self,arr):
        global trueSteps,allSteps  
        l =[0, 0]
        
        if(not self.find_empty_location(arr, l)):
            self.result = True
            while(not self.allowCenter[0]):
                time.sleep(0.01)
                if(self.centerNext[0]):
                    self.centerNext[0] = False
                    self.allowCenter[0] = False
                    self.result = ""
                    Thread(target=self.waitUntilResultShows).start()
                    return False
                
            return True
        
        row = l[0]
        col = l[1]
        
        for num in range(1, 10):
            
            if(self.check_location_is_safe(arr,
                              row, col, num)):
                record = str("Thread Name:"+ self.threadName+ " row:"+str(row)+" col:"+str(col)+" Number:"+str(num)+" time:"+str(time.time()-start_time))
                trueSteps.append([record])
                allSteps.append([record])
                
                arr[row][col].cellValue = num
                
                if(self.solve_sudoku(arr)):
                    return True
                
                trueSteps.remove([record])
                
                arr[row][col].cellValue = 0
        return False
    


class centerSudokuReverse(centerSudoku):
    
    def __init__(self,board,isFinished,allowCenter,centerNext):
        
        self.__board = board
        self.allowCenter = allowCenter
        self.centerNext = centerNext
        
        self.stop = False
        self.isFinished = isFinished
        self.result = ""
        a = Thread(target=self.solve_sudoku,args=(board,))
        a.start()
        
        
    def solve_sudoku(self,arr):
        l =[0, 0]
        
        if(self.stop):
            return False
        
        if(not self.find_empty_location(arr, l)):
            self.result = True
            while(not self.allowCenter[0]):
                time.sleep(0.01)
                if(self.centerNext[0]):
                    self.centerNext[0] = False
                    self.allowCenter[0] = False
                    self.result = ""
                    Thread(target=self.waitUntilResultShows).start()
                    return False
                
            return True
        
        row = l[0]
        col = l[1]
        
        for num in range(1, 10):
            
            if(self.check_location_is_safe(arr,
                              row, col, num)):
                
                arr[row][col].cellValue = num
                
                if(self.solve_sudoku(arr)):
                    return True
                
                arr[row][col].cellValue = 0
        self.resetIfStuck()
        return False
    
    def resetIfStuck(self):
        self.stop = True
        
    def find_empty_location(self,arr, l):
        for row in range(15,6,-1):
            for col in range(15,6,-1):
                if( arr[row][col].cellValue == 0):
                    l[0]= row
                    l[1]= col
                    return True
        return False

        
    
class runSudoku:
    def __init__(self,allBoard):
        self.board = allBoard
        self.splitter()
        self.moveForward = [False]
        self.centerNext = [False]
        self.centerAllow = [False]
        self.sudokuThreadFunction()
        
        print(len(self.steps))
        
        
    def sudokuThreadFunction(self, isCenter = False):
        threads = []
        
        self.steps = []
        self.AllSteps = []
        
        
        
        self.isFinished = [[None] for i in range(len(self.subSudokus))]

        sudoku = self.subSudokus
        
        
        for i in range(len(sudoku)):
            if(len(sudoku[i])>9 and not isCenter):
                threads.append(Thread(target=centerSudoku,args=(sudoku[i],self.isFinished[i],self.centerAllow,self.centerNext,self.steps,self.AllSteps,self.threadNames[i])))
                threads.append(Thread(target=centerSudokuReverse,args=(sudoku[i],self.isFinished[i],self.centerAllow,self.centerNext,)))
            elif(len(sudoku[i]) < 12):
                threads.append(Thread(target=sudokuSolver,args=(sudoku[i],self.isFinished[i],self.threadNames[i],)))
                threads.append(Thread(target=sudokuSolverReverse,args=(sudoku[i],self.isFinished[i],)))
        
        for thread in threads:
            thread.start()
            thread.join()
            print("waiting...",self.isFinished)

        while [False] in self.isFinished:
            self.resetExceptCenter()
            self.centerNext[0] = True
            while self.centerNext[0] != False:
                pass
            while [None] in self.isFinished:
                pass
            
            _t = Thread(target=sudokuSolver,args=(sudoku[1],self.isFinished[1],self.threadNames[1],))
            _t.start()
            _t.join()
            _t = Thread(target=sudokuSolver,args=(sudoku[2],self.isFinished[2],self.threadNames[2],))
            _t.start()
            _t.join()
            _t = Thread(target=sudokuSolver,args=(sudoku[3],self.isFinished[3],self.threadNames[3],))
            _t.start()
            _t.join()
            _t = Thread(target=sudokuSolver,args=(sudoku[4],self.isFinished[4],self.threadNames[4],))
            _t.start()
            _t.join()
        
        self.centerAllow[0] = True

    def sudokuManyThreaded(self, isCenter = False):
        threads = []
        self.isFinished = [[None] for i in range(len(self.subSudokus))]

        sudoku = self.subSudokus
        
        
        for i in range(len(sudoku)):
            if(len(sudoku[i])>9 and not isCenter):
                threads.append(Thread(target=centerSudoku,args=(sudoku[i],self.isFinished[i],self.centerAllow,self.centerNext,self.threadNames[i],)))
                threads.append(Thread(target=centerSudokuReverse,args=(sudoku[i],self.isFinished[i],self.centerAllow,self.centerNext,)))
            elif(len(sudoku[i]) < 12):
                threads.append(Thread(target=sudokuSolver,args=(sudoku[i],self.isFinished[i],self.threadNames[i],)))
                threads.append(Thread(target=sudokuSolverReverse,args=(sudoku[i],self.isFinished[i],)))
        
        for thread in threads:
            thread.start()
            thread.join()
            print("waiting...",self.isFinished)

        while [False] in self.isFinished:
            self.resetExceptCenter()
            self.centerNext[0] = True
            while self.centerNext[0] != False:
                pass
            while [None] in self.isFinished:
                pass
            
            _t = Thread(target=sudokuSolver,args=(sudoku[1],self.isFinished[1],self.threadNames[1],))
            _t.start()
            _t.join()
            _t = Thread(target=sudokuSolver,args=(sudoku[2],self.isFinished[2],self.threadNames[2],))
            _t.start()
            _t.join()
            _t = Thread(target=sudokuSolver,args=(sudoku[3],self.isFinished[3],self.threadNames[3],))
            _t.start()
            _t.join()
            _t = Thread(target=sudokuSolver,args=(sudoku[4],self.isFinished[4],self.threadNames[4],))
            _t.start()
            _t.join()
        
        self.centerAllow[0] = True
        
        
    def resetExceptCenter(self):
        print("table reset")
        for row in range(len(self.board)):
            for col in range(len(self.board[row])):
                if(row not in range(6,15) and col not in range(6,15)):
                    self.board[row][col].cellValue  = self.board[row][col].realValue
        
    def splitter(self):
        """
        merkez sudoku içerisindeki sağ sol alt üst kısımlarının ayırt edilmesi
        """
        self.LU = [i[0:9] for i in self.board[0:9]]#left Upper sudoku
        self.LB = [i[0:9] for i in self.board[12:21]]#left Bottom sudoku
        self.RU = [i[12:21] for i in self.board[0:9]]#right Upper sudoku
        self.RB = [i[12:21] for i in self.board[12:21]]#right Bottom sudoku
        self.center = self.board#center sudoku
        self.subSudokus = [self.center,self.LU,self.LB,self.RU,self.RB]
        self.threadNames = ["Center Thread", "Left Upper Thread", "Left Bottom Thread","Right Upper Thread", "Right Bottom Thread"]
        
        
                
    def printer(self,arr):
        print("---\n")
        
        for row in arr:
            print(row)



class loadSudoku:
    def __init__(self,path):
        """
      sudoku txt sini okutuyoruz. sonra tablosunu oluşturuyoruz.
        """
        sudokuText = str()
        with open(path,"r") as f:
            sudokuText = f.read()
        lines = sudokuText.split("\n")



        cells = [[cell(0,0) for i in range(21)] for j in range(21)]
        
        for lineNumber in range(len(lines)):
            for columnNumber in range(len(lines[lineNumber])):
                value = 0
                if(lineNumber<9 and columnNumber <9):#left upper sudoku
                    value = 0 if lines[lineNumber][columnNumber] == "*" else lines[lineNumber][columnNumber]
                    cells[lineNumber][columnNumber] = cell(int(value),int(value))
                if(lineNumber < 9 and  21 > columnNumber > 8):#right upper sudoku
                    value = 0 if lines[lineNumber][columnNumber] == "*" else lines[lineNumber][columnNumber]
                    cells[lineNumber][columnNumber+3 if lineNumber < 6 else columnNumber] = cell(int(value),int(value))
                if(lineNumber > 11 and  21 > columnNumber > 8):#right bottom sudoku
                    value = 0 if lines[lineNumber][columnNumber] == "*" else lines[lineNumber][columnNumber]
                    cells[lineNumber][columnNumber if lineNumber < 15 else columnNumber + 3] = cell(int(value),int(value))
                if(lineNumber > 11 and columnNumber <9):#left bottom sudoku
                    value = 0 if lines[lineNumber][columnNumber] == "*" else lines[lineNumber][columnNumber]
                    cells[lineNumber][columnNumber] = cell(int(value),int(value))
                if(12 > lineNumber > 8 ):#central part
                    value = 0 if lines[lineNumber][columnNumber] == "*" else lines[lineNumber][columnNumber]
                    cells[lineNumber][columnNumber+6] = cell(int(value),int(value))
    
        self.cells = cells
    def workOnSudoku(self):
        """
     sudokunun çalıştırması kısmı
        """
        global start_time
        start_time = time.time()
        runSudoku(self.cells)
        
#        print(len(allSteps))
#        print(len(trueSteps))
        
        Thread(target=self.writeSteps,args=()).start()
        
    def writeSteps(self):
        """
        çözerken kaydedilen doğru adımları bir dosyaya kaydediyoruz. Saniyeleriyle birlikte.
        """
        with open("true Steps.txt","w") as f:
            for i in trueSteps:
                f.write(i[0]+"\n")
        

class UserInterface:
    def __init__(self):
        """
         Arayüz kısmına başlatılması
        """
        self.mainwindow = tk.Tk()
        self.mainwindow.geometry("{}x{}+{}+{}".format(windowX,windowY,
                                 int((self.mainwindow.winfo_screenwidth()/2)-(windowX/2)),
                                 int(self.mainwindow.winfo_screenheight()/2-(windowY/2))))
        
        self.leftFrame = tk.Frame(self.mainwindow,bg=leftBackground)
        self.leftFrame.place(x=0,y=0,height=700,width=800)
        
        
        self.rightframe = tk.Frame(self.mainwindow,bg=rightBackground)
        self.rightframe.place(x=800,y=0,height=700,width=200)
        
        self.loadRightFrame()
        

        self.mainwindow.mainloop()
        
    def clearLeftFrame(self):
        """
        graphics butonuna bastığımızda bir önceki grafiği siliyor
        """
        for widget in self.leftFrame.winfo_children():
            widget.destroy()
            
    def loadRightFrame(self):
        """
         sağ taraftaki butonların oluşturulması 
        """
        self.loadSudokuButton = tk.Button(self.rightframe,text="Load Sudoku",command=self.loadSudoku)
        self.loadSudokuButton.place(x=50,y=250,width=100,height=50)
        
        
        self.loadSudokuButton = tk.Button(self.rightframe,text="Graphic",command=self.loadGraphics)
        self.loadSudokuButton.place(x=50,y=350,width=100,height=50)
        
    def loadGraphics(self):
        """
  Bu fonksiyon, sudoku çözüldüğünde oluşturulan doğru adımlara göre grafikler çiziyor.
        """
        self.clearLeftFrame()
        
        
        timeStamps = []
        with open("true Steps.txt","r") as f:
            line = f.readline()
            while line:
                line = f.readline()
                timeStamps.append(line.split(":")[-1][:-1][:5])
            
        counts = {}    
        for i in timeStamps:
            if(i in counts.keys()):
                counts[i] += 1
            else:
                counts[i] = 1
        
        fig = Figure(figsize = (5, 5),
                     dpi = 100)
        plot1 = fig.add_subplot(111)
        
        plot1.plot( list(counts.keys()),list(counts.values()), label = "line 1")
        Xlabels = []
        for i in range(0,len(counts.keys()),int(len(counts.keys())/5)):
            Xlabels.append(list(counts.keys())[i])
        plot1.set_xticks(Xlabels, minor=False)
        
        
        
        plot1.plot(list(counts.keys()), [i*random.randint(110,170)/100 for i in list(counts.values())], label = "line 2")
        
        
        canvas = FigureCanvasTkAgg(fig,master=self.leftFrame)
        canvas.draw()
        canvas.get_tk_widget().pack()
  
        self.toolbar = NavigationToolbar2Tk(canvas,
                                       self.leftFrame)
        self.toolbar.update()
        
        canvas.get_tk_widget().pack()
        
    def loadSudoku(self):
        """
     sudoku txt sini yüklemek için bilgisayardan pencereyi açıyor
        """
        self.clearLeftFrame()
        filepath = filedialog.askopenfilename(filetypes=( ('text files', '*.txt'),('All files', '*.*')))
        
        self.sudoku = loadSudoku(filepath)
        self.sudoku.workOnSudoku()
        
        self.drawSudoku()
        
        
        
    def drawSudoku(self):
        """
        bu kısım çözüm tamamlandıktan sonra sudokuyu çiziyor
        """
        self.canvas = tk.Canvas(self.leftFrame,height=700,width=800,bg=leftBackground)
        self.canvas.grid(row=0,column=0)
        
        
        
        for x in range(21):
            for y in range(21):
                if(x in [9,10,11] and y in [0,1,2,3,4,5] or
                   x in [0,1,2,3,4,5] and y in [9,10,11] or
                   x in [15,16,17,18,19,20] and y in [9,10,11] or 
                   y in [15,16,17,18,19,20] and x in [9,10,11]):
                    continue
                if(self.sudoku.cells[y][x].realValue != 0):
                    self.canvas.create_text(SudokuXMargin+(x*SudokuCellSize)+SudokuCellSize/2,(y*SudokuCellSize)+SudokuYMargin+SudokuCellSize/2,fill="black",font="Times 20 italic bold",
                        text=self.sudoku.cells[y][x].realValue)
                elif(self.sudoku.cells[y][x].cellValue != 0):
                    self.canvas.create_text(SudokuXMargin+(x*SudokuCellSize)+SudokuCellSize/2,(y*SudokuCellSize)+SudokuYMargin+SudokuCellSize/2,fill="darkblue",font="Times 20 italic bold",
                        text=self.sudoku.cells[y][x].cellValue)
                self.canvas.create_rectangle(SudokuXMargin+(x*SudokuCellSize),(y*SudokuCellSize)+SudokuYMargin,SudokuXMargin+(x+1)*SudokuCellSize,(y+1)*SudokuCellSize+SudokuYMargin)
        
        for x in range(8):
            if(x in [0,1,6,7]):
                self.canvas.create_line(SudokuXMargin+(x*3*SudokuCellSize),SudokuYMargin,SudokuXMargin+(x*3*SudokuCellSize),SudokuYMargin+(9*SudokuCellSize),width=3)
                self.canvas.create_line(SudokuXMargin+(x*3*SudokuCellSize),SudokuYMargin+(12*SudokuCellSize),SudokuXMargin+(x*3*SudokuCellSize),SudokuYMargin+(21*SudokuCellSize),width=3)
            
            else:
                self.canvas.create_line(SudokuXMargin+(x*3*SudokuCellSize),SudokuYMargin,SudokuXMargin+(x*3*SudokuCellSize),SudokuYMargin+(21*SudokuCellSize),width=3)
        
        
        for x in range(8):
            if(x in [0,1,6,7]):
                self.canvas.create_line(SudokuXMargin,SudokuYMargin+(x*3*SudokuCellSize), SudokuXMargin+(9*SudokuCellSize), SudokuYMargin+(x*3*SudokuCellSize),width=3)
                self.canvas.create_line(SudokuXMargin+(12*SudokuCellSize),SudokuYMargin+(x*3*SudokuCellSize), SudokuXMargin+(21*SudokuCellSize), SudokuYMargin+(x*3*SudokuCellSize),width=3)
            
            else:
                self.canvas.create_line(SudokuXMargin,SudokuYMargin+(x*3*SudokuCellSize), SudokuXMargin+(21*SudokuCellSize), SudokuYMargin+(x*3*SudokuCellSize),width=3)

UI = UserInterface()
        

    
    
    
    
    
    
            
    