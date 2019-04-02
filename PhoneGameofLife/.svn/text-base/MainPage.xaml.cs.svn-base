//---------------------------------------------------------------------
// <copyright file="MainPage.xaml.cs" company="Microsoft">
// Copyright (c) Microsoft Corporation. All rights reserved.
// </copyright>
// <summary>
// Use of this source code is subject to the terms of your Windows Mobile
// Software Shared Source Premium Derivatives License Form or other
// applicable Microsoft license agreement for the software. If you did not
// accept the terms of such a license, you are not authorized to use this
// source code.
// </summary>
//---------------------------------------------------------------------

using System;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Shapes;
using System.Windows.Threading;
using Microsoft.Phone.Controls;
using System.Collections.Generic;
using System.Threading;

namespace PhoneGameofLife
{
    public partial class MainPage : PhoneApplicationPage
    {
        #region Private Members
        private const int NUM_ROWS = 20;
        private Rectangle[,] grid = new Rectangle[NUM_ROWS, NUM_ROWS];
        private Brush darkBrush;
        private Brush lightBrush;
        private DispatcherTimer dt = new DispatcherTimer();
        private Queue<Boolean[,]> nextgrids = new Queue<Boolean[,]>();
        private int minRow, maxRow, minCol, maxCol;
        private Thread _modelThread;
        private long _runningtimetotal;
        private long _runningsizetotal;
        private long _numsamples;
        private bool _isfirst;
        private List<int> _timedata;
        private List<int> _queuedata;

        #endregion

        #region Constructor

        /// <summary>
        /// Constructor for MainPage. Creates grid control with all dead blocks, with dimensions
        /// of NUM_ROWS x NUM_ROWS. This also creates the timer which animates the game, and sets
        /// the text for the sliders used in the application.
        /// </summary>
        public MainPage()
        {
            InitializeComponent();
            for (int i = 0; i < NUM_ROWS; i++)
            {
                this.mainPanel.RowDefinitions.Add(new RowDefinition());
                this.mainPanel.ColumnDefinitions.Add(new ColumnDefinition());
            }
            lightBrush = Application.Current.Resources["PhoneAccentBrush"] as Brush;
            darkBrush = Application.Current.Resources["PhoneBackgroundBrush"] as Brush;

            for (int i = 0; i < NUM_ROWS; i++)
            {
                for (int j = 0; j < NUM_ROWS; j++)
                {
                    Rectangle darkRect = new Rectangle();
                    darkRect.Fill = darkBrush;
                    Grid.SetColumn(darkRect, j);
                    Grid.SetRow(darkRect, i);
                    Grid.SetColumnSpan(darkRect, 1);
                    Grid.SetRowSpan(darkRect, 1);
                    this.mainPanel.Children.Add(darkRect);
                    darkRect.MouseLeftButtonUp += new MouseButtonEventHandler(MainPanelCell_MouseLeftButtonUp);
                    grid[i, j] = darkRect;
                }
            }

            this.dt.Interval = new TimeSpan(0, 0, 0, 0, 500);
            this.dt.Tick += new EventHandler(dt_Tick);
            txtSpeed.Text = ((int)slSpeed.Value).ToString();
            txtZoom.Text = String.Format("{0:0.##}", this.slZoom.Value) + "x";
            this.minRow = NUM_ROWS + 1;
            this.minCol = NUM_ROWS + 1;
            this.maxRow = -1;
            this.maxCol = -1;
        }

        #endregion

        #region Game Logic

        /// <summary>
        /// This method sets each rectangle's fill property as indicated by the next configuration determined 
        /// earlier by GetNextTurn. It then calls GetNextTurn in order to prepare for the next turn in the game.
        /// </summary>
        private void RunTurn()
        {
            while (this.nextgrids.Count == 0)
            {
                Thread.SpinWait(5);
            }
            Boolean[,] nextgrid = this.nextgrids.Dequeue();
            Boolean anyLive = false;
            for (int i = this.minRow; i < this.maxRow; i++)
            {
                for (int j = this.minCol; j < this.maxCol; j++)
                {
                    if (nextgrid[i, j])
                    {
                        grid[i, j].Fill = this.lightBrush;
                        anyLive = true;
                    }
                    else
                    {
                        grid[i, j].Fill = this.darkBrush;
                    }
                }
            }
            if (!anyLive)
            {
                dt.Stop();
            }
            AdjustBoundaries(nextgrid);
        }

        /// <summary>
        /// This method determines what the next configuration of the grid is, using the current configuration.
        /// This is run in between turns, so that it doesn't delay each turn from being displayed when possible.
        /// </summary>
        private void GetTurns(Boolean[,] prevGrid)
        {
            //Rectangle[,] prevGrid = new Rectangle[NUM_ROWS, NUM_ROWS];
            Boolean[,] nextGrid = new Boolean[NUM_ROWS, NUM_ROWS];
            while (dt.IsEnabled)
            {
                while (this.nextgrids.Count > 5)
                {
                    Thread.Sleep(50);
                }
                GetTurn(ref prevGrid, ref nextGrid);
            }
        }

        private void GetTurn(ref Boolean[,] prevGrid, ref Boolean[,] nextGrid)
        {
            for (int i = this.minRow; i < this.maxRow; i++)
            {
                for (int j = this.minCol; j < this.maxCol; j++)
                {
                    int liveNeighbors = 0;
                    // Determine number of neighbors.
                    if (i < (NUM_ROWS - 1) && prevGrid[i + 1, j]) liveNeighbors++;
                    if (i > 0 && prevGrid[i - 1, j]) liveNeighbors++;
                    if (i < (NUM_ROWS - 1) && j < (NUM_ROWS - 1) && prevGrid[i + 1, j + 1]) liveNeighbors++;
                    if (j > 0 && i < (NUM_ROWS - 1) && prevGrid[i + 1, j - 1]) liveNeighbors++;
                    if (j < (NUM_ROWS - 1) && prevGrid[i, j + 1]) liveNeighbors++;
                    if (j > 0 && prevGrid[i, j - 1]) liveNeighbors++;
                    if (i > 0 && j < (NUM_ROWS - 1) && prevGrid[i - 1, j + 1]) liveNeighbors++;
                    if (i > 0 && j > 0 && prevGrid[i - 1, j - 1]) liveNeighbors++;

                    // Use number of neighbors to determine liveness of cell.
                    if (prevGrid[i, j])
                    {
                        nextGrid[i, j] = (liveNeighbors == 2 || liveNeighbors == 3);
                    }
                    else
                    {
                        nextGrid[i, j] = (liveNeighbors == 3);
                    }
                }
            }
            this.nextgrids.Enqueue(nextGrid);
            prevGrid = (Boolean[,]) nextGrid.Clone();
        }

        #endregion

        #region Preset Patterns
        /// <summary>
        /// This method creates the premade pattern blinker in the middle of the board.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void mnuBlinker_Click(object sender, EventArgs e)
        {
            if (!dt.IsEnabled)
            {
                for (int i = 0; i < NUM_ROWS; i++)
                {
                    for (int j = 0; j < NUM_ROWS; j++)
                    {
                        if (i == 10 && (j == 9 || j == 10 || j == 11))
                        {
                            this.grid[i, j].Fill = lightBrush;
                        }
                        else
                        {
                            this.grid[i, j].Fill = darkBrush;
                        }
                    }
                }
            }
        }

        /// <summary>
        /// This method creates the premade pattern toad in the middle of the board.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void mnuToad_Click(object sender, EventArgs e)
        {
            if (!dt.IsEnabled)
            {
                for (int i = 0; i < NUM_ROWS; i++)
                {
                    for (int j = 0; j < NUM_ROWS; j++)
                    {
                        if ((i == 10 && (j == 9 || j == 10 || j == 11)) || (i == 11 && (j == 8 || j == 9 || j == 10)))
                        {
                            this.grid[i, j].Fill = lightBrush;
                        }
                        else
                        {
                            this.grid[i, j].Fill = darkBrush;
                        }
                    }
                }
            }
        }

        /// <summary>
        /// This method creates the premade pattern Beacon in the middle of the board.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void mnuBeacon_Click(object sender, EventArgs e)
        {
            if (!dt.IsEnabled)
            {
                for (int i = 0; i < NUM_ROWS; i++)
                {
                    for (int j = 0; j < NUM_ROWS; j++)
                    {
                        if ((i == 9 && (j == 8 || j == 9)) || (i == 10 && (j == 8 || j == 9)) || (i == 11 && (j == 10 || j == 11)) || (i == 12 && (j == 10 || j == 11)))
                        {
                            this.grid[i, j].Fill = lightBrush;
                        }
                        else
                        {
                            this.grid[i, j].Fill = darkBrush;
                        }
                    }
                }
            }
        }

        /// <summary>
        /// This pattern makes a premade flier in the upper-lefthand corner of the board, flying towards
        /// the opposite corner.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void mnuFlier_Click(object sender, EventArgs e)
        {
            if (!dt.IsEnabled)
            {
                for (int i = 0; i < NUM_ROWS; i++)
                {
                    for (int j = 0; j < NUM_ROWS; j++)
                    {
                        if ((i == 0 && j == 1) || (i == 1 && j == 2) || (i == 2 && (j == 0 || j == 1 || j == 2)))
                        {
                            this.grid[i, j].Fill = lightBrush;
                        }
                        else
                        {
                            this.grid[i, j].Fill = darkBrush;
                        }
                    }
                }
            }
        }

        private void mnuPulsar_Click(object sender, EventArgs e)
        {
            if (!dt.IsEnabled)
            {
                for (int i = 0; i < NUM_ROWS; i++)
                {
                    for (int j = 0; j < NUM_ROWS; j++)
                    {
                        if (((i == 4 || i == 9 || i == 11 || i == 16) && (j == 5 || j == 6 || j == 7 || j == 11 || j == 12 || j == 13))
                            || ((i == 6 || i == 7 || i == 8 || i == 12 || i == 13 || i == 14) && (j == 3 || j == 8 || j == 10 || j == 15)))
                        {
                            this.grid[i, j].Fill = lightBrush;
                        }
                        else
                        {
                            this.grid[i, j].Fill = darkBrush;
                        }
                    }
                }
            }
        }

        #endregion

        #region Helper Methods

        private void InitBoundaries()
        {
            int nextMinRow = NUM_ROWS;
            int nextMaxRow = -1;
            int nextMinCol = NUM_ROWS;
            int nextMaxCol = -1;
            for (int row = 0; row < NUM_ROWS; row++)
            {
                for (int col = 0; col < NUM_ROWS; col++)
                {
                    if (this.grid[row, col].Fill.Equals(this.lightBrush))
                    {
                        if (row == 0)
                        {
                            nextMinRow = 0;
                        }
                        else if (row <= nextMinRow)
                        {
                            nextMinRow = row - 1;
                        }
                        if (row == NUM_ROWS - 1)
                        {
                            nextMaxRow = NUM_ROWS;
                        }
                        else if (row >= nextMaxRow)
                        {
                            nextMaxRow = row + 2;
                        }
                        if (col == 0)
                        {
                            nextMinCol = 0;
                        }
                        else if (col <= nextMinCol)
                        {
                            nextMinCol = col - 1;
                        }
                        if (col == NUM_ROWS - 1)
                        {
                            nextMaxCol = NUM_ROWS;
                        }
                        else if (col >= nextMaxCol)
                        {
                            nextMaxCol = col + 2;
                        }
                    }
                }
            }
            this.minRow = nextMinRow;
            this.maxRow = nextMaxRow;
            this.minCol = nextMinCol;
            this.maxCol = nextMaxCol;
        }

        private void AdjustBoundaries(Boolean[,] nextGrid)
        {
            int nextMinRow = NUM_ROWS;
            int nextMaxRow = -1;
            int nextMinCol = NUM_ROWS;
            int nextMaxCol = -1;
            for (int row = this.minRow; row < this.maxRow; row++)
            {
                for (int col = this.minCol; col < this.maxCol; col++)
                {
                    if (nextGrid[row, col])
                    {
                        if (row == 0)
                        {
                            nextMinRow = 0;
                        }
                        else if (row <= nextMinRow)
                        {
                            nextMinRow = row - 1;
                        }
                        if (row == NUM_ROWS - 1)
                        {
                            nextMaxRow = NUM_ROWS;
                        }
                        else if (row >= nextMaxRow)
                        {
                            nextMaxRow = row + 2;
                        }
                        if (col == 0)
                        {
                            nextMinCol = 0;
                        }
                        else if (col <= nextMinCol)
                        {
                            nextMinCol = col - 1;
                        }
                        if (col == NUM_ROWS - 1)
                        {
                            nextMaxCol = NUM_ROWS;
                        }
                        else if (col >= nextMaxCol)
                        {
                            nextMaxCol = col + 2;
                        }
                    }
                }
            }
            this.minRow = nextMinRow;
            this.maxRow = nextMaxRow;
            this.minCol = nextMinCol;
            this.maxCol = nextMaxCol;
        }

        private Boolean[,] ConvertToBool(Rectangle[,] grid) {
            Boolean[,] next = new Boolean[NUM_ROWS, NUM_ROWS];
            for (int i = 0; i < NUM_ROWS; i++)
            {
                for (int j = 0; j < NUM_ROWS; j++)
                {
                    next[i, j] = grid[i, j].Fill.Equals(this.lightBrush);
                }
            }
            return next;
        }

        #endregion

        #region Event Handlers

        /// <summary>
        /// This method is triggered when the zoom slider's value is changed. This method scales the
        /// grid according to the value of the slider, and sets the text next to the slider to indicate
        /// this change.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void slZoom_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
        {
            if (slZoom != null)
            {
                txtZoom.Text = String.Format("{0:0.##}", this.slZoom.Value) + "x";
                this.mainPanel.Width = 300 + 100 * this.slZoom.Value;
                this.mainPanel.Height = 300 + 100 * this.slZoom.Value;
            }
        }

        private void btnStep_Click(object sender, EventArgs e)
        {
            Boolean[,] tmp = new Boolean[NUM_ROWS, NUM_ROWS];
            Boolean[,] next = ConvertToBool(this.grid);
            GetTurn(ref next, ref tmp);
            this.nextgrids.Enqueue(next);
            RunTurn();
        }

        /// <summary>
        /// This method is triggered when the slider indicating the speed is changed. The timer's frequency is then set
        /// to the value of the slider in Hz. The text next to the slider is then updated to reflect this change.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void slSpeed_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
        {
            if (slSpeed != null)
            {
                dt.Interval = new TimeSpan(0, 0, 0, (int)(1.0 / ((double)slSpeed.Value)), (int)(1000.0 * 1.0 / ((double)slSpeed.Value)));
                txtSpeed.Text = ((int)slSpeed.Value).ToString();
            }
        }

        /// <summary>
        /// This method starts the game. It first checks that there is a cell alive. If not, the 
        /// game is not started. Otherwise, the next turn is obtained (but not shown), and the timer
        /// is started.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void btnStart_Click(object sender, EventArgs e)
        {
            bool isEmpty = true;
            for (int i = 0; i < NUM_ROWS; i++)
            {
                for (int j = 0; j < NUM_ROWS; j++)
                {
                    if (this.grid[i, j].Fill != this.darkBrush)
                    {
                        isEmpty = false;
                        break;
                    }
                }
            }
            if (!isEmpty)
            {
                this._runningtimetotal = 0;
                this._numsamples = 0;
                this._isfirst = true;
                this._timedata = new List<int>();
                this._queuedata = new List<int>();
                Boolean[,] currGrid = ConvertToBool(this.grid);
                InitBoundaries();
                ThreadStart start = delegate { GetTurns(currGrid); };
                this._modelThread = new Thread(start);
                dt.Start();
                this._modelThread.Start();
            }
        }

        /// <summary>
        /// This method pauses the timer which controls the game, leaving all of the cells as they were.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void btnPause_Click(object sender, EventArgs e)
        {
            dt.Stop();
        }

        /// <summary>
        /// This method sets all cells to dead, thus clearing the board.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void btnClear_Click(object sender, EventArgs e)
        {
            this.dt.Stop();
            for (int i = 0; i < NUM_ROWS; i++)
            {
                for (int j = 0; j < NUM_ROWS; j++)
                {
                    this.grid[i, j].Fill = this.darkBrush;
                }
            }
        }

        /// <summary>
        /// Method which is attached to the event triggered by the timer. This in turn runs RunTurn, in order
        /// to change the configuration to the next configuration.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void dt_Tick(object sender, EventArgs e)
        {
            DateTime dt = DateTime.Now;
            RunTurn();
            int interval = DateTime.Now.Subtract(dt).Milliseconds;
            int qCount = this.nextgrids.Count;
            if (!this._isfirst)
            {
                this._timedata.Add(interval);
                this._queuedata.Add(qCount);
                this._runningtimetotal += interval;
                this._runningsizetotal += qCount;
                this._timedata.Sort();
                this._queuedata.Sort();
                this.txtTest.Text = "Time Min: " + this._timedata[0] + "; Max: " + this._timedata[this._timedata.Count - 1] + "; Average: " + String.Format("{0:0.000}", ((double)this._runningtimetotal / (double)this._timedata.Count)) + "\n";
                this.txtTest.Text += "Queue Min: " + this._queuedata[0] + "; Max: " + this._queuedata[this._queuedata.Count - 1] + "; Average: " + String.Format("{0:0.000}", ((double) this._runningsizetotal / (double) this._queuedata.Count));
            }
            else
            {
                this._isfirst = false;
            }
        }

        /// <summary>
        /// Method which is triggered when a user clicks on one of the cells in the grid. This will change
        /// the color of the cell if the timer is disabled. Otherwise, nothing will happen, in order to ensure
        /// no strange behavior during runtime.
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void MainPanelCell_MouseLeftButtonUp(object sender, EventArgs e)
        {
            if (!dt.IsEnabled)
            {
                Rectangle sendRect = (Rectangle)sender;
                if (sendRect.Fill.Equals(darkBrush))
                {
                    sendRect.Fill = lightBrush;
                }
                else
                {
                    sendRect.Fill = darkBrush;
                }
            }
        }

        #endregion
    }
}
