The accepted [answer](https://stackoverflow.com/a/75297887/5438626) is excellent, but I would be remiss if I didn't mention how `DataGridView` might save you a lot of legwork by automatically configuring itself based on your `Destination` class.

[![screenshot][1]][1]

***
>I have a class [...] called Destination. The output has to be "Name Distance Price".

    public class Destination
    {
        [ReadOnly(true)]
        public string Name { get; set; } = string.Empty;

        [ReadOnly(true)]
        public int Distance { get; set; }

        [ReadOnly(false)]
        public decimal Price { get; set; }

        public override string ToString() =>
            string.Format($"{Name} {Distance} km {Price} $");
    }
***
> I want to sort them by the distance or the price. 

    private void sortByHeader(object? sender, DataGridViewCellMouseEventArgs e)
    {
        Destination[] tmp;
        if (!e.ColumnIndex.Equals(-1))
        {
            var column = dataGridView.Columns[e.ColumnIndex].Name;
            if (column.Equals(_prevColumn))
            {
                _highToLow = !_highToLow;
            }
            else
            {
                _highToLow = false;
                _prevColumn = column;
            }
            switch (column)
            {
                case nameof(Destination.Name):
                    tmp = _highToLow ?
                        Destinations.OrderByDescending(_ => _.Name).ToArray() :
                        Destinations.OrderBy(_ => _.Name).ToArray();
                    break;
                case nameof(Destination.Distance):
                    tmp = _highToLow ?
                        Destinations.OrderByDescending(_ => _.Distance).ToArray() :
                        Destinations.OrderBy(_ => _.Distance).ToArray();
                    break;
                case nameof(Destination.Price):
                    tmp = _highToLow ?
                        Destinations.OrderByDescending(_ => _.Price).ToArray() :
                        Destinations.OrderBy(_ => _.Price).ToArray();
                    break;
                default:
                    return;
            }
            Destinations.Clear();
            foreach (var destination in tmp)
            {
                Destinations.Add(destination);
            }
        }
    }
    string? _prevColumn = null;
    bool _highToLow = false;
***
The method that loads the main form configures the `DataGridView` using the `Destination` class as a template and attaches the `dataGridView.ColumnHeaderMouseClick` event to sort the data. The `DataSource` of the data grid is set to `Destinations` which is a `BindingList<Destination>`.

    public partial class MainForm : Form
    {
        public MainForm() =>InitializeComponent();
        private readonly BindingList<Destination> Destinations= new BindingList<Destination>();
        protected override void OnLoad(EventArgs e)
        {
            base.OnLoad(e);
            dataGridView.RowTemplate.Height = 60;
            // Add desinations interactively?
            dataGridView.AllowUserToAddRows= false;
            dataGridView.DataSource= Destinations;

            #region F O R M A T    C O L U M N S
            Destinations.Add(new Destination()); // <= Auto-generate columns
            dataGridView.Columns[nameof(Destination.Name)].AutoSizeMode = DataGridViewAutoSizeColumnMode.Fill;
            dataGridView.Columns[nameof(Destination.Distance)].AutoSizeMode = DataGridViewAutoSizeColumnMode.AllCells;
            dataGridView.Columns[nameof(Destination.Price)].AutoSizeMode = DataGridViewAutoSizeColumnMode.AllCells;
            dataGridView.Columns[nameof(Destination.Price)].DefaultCellStyle.Format = "F2";
            Destinations.Clear();
            #endregion F O R M A T    C O L U M N S

            dataGridView.ColumnHeaderMouseClick += sortByHeader;
            addDestinationsToExample();
        }
        .
        .
        .
    }

Consecutive clicks on the same header will perform alternate low-to-high or high-to-low.

***
>Now I want to add destinations with a name, the distance to that location and the price for that trip...

    private void addDestinationsToExample()
    {
        Destinations.Add(new Destination 
        { 
            // * Had to make the Price sort different from Distance sort!
            Name = "London - VIP", 
            Distance= 100,
            Price= 1200,
        });
        Destinations.Add(new Destination 
        { 
            Name = "Berlin",
            Distance= 400,
            Price= 800,
        });
        Destinations.Add(new Destination 
        { 
            Name = "Paris",
            Distance= 200,
            Price= 400,
        });
        Destinations.Add(new Destination 
        { 
            Name = "Madrid",
            Distance= 150,
            Price= 300,
        });
    }

  I hope this adds to your toolbox of possible ways to achieve your objective.

  [1]: https://i.stack.imgur.com/m1xCc.png