# FSharp.ViewModule

## Example[#](https://riptutorial.com/fsharp/example/27305/fsharp-viewmodule#example)

Our demo app consists of a scoreboard. The score model is an immutable record. The scoreboard events are contained in a Union Type.

```fsharp
namespace Score.Model

type Score = { ScoreA: int ; ScoreB: int }    
type ScoringEvent = IncA | DecA | IncB | DecB | NewGame
```

Changes are propagated by listening for events and updating the view model accordingly. Instead of adding members to the model type, as in OOP, we declare a separate module to host the allowed operations.

```fsharp
[<CompilationRepresentation(CompilationRepresentationFlags.ModuleSuffix)>]
module Score =
    let zero = {ScoreA = 0; ScoreB = 0}
    let update score event =
        match event with
        | IncA -> {score with ScoreA = score.ScoreA + 1}
        | DecA -> {score with ScoreA = max (score.ScoreA - 1) 0}
        | IncB -> {score with ScoreB = score.ScoreB + 1}
        | DecB -> {score with ScoreB = max (score.ScoreB - 1) 0}
        | NewGame -> zero 
```

Our view model derives from `EventViewModelBase<'a>`, which has a property `EventStream` of type `IObservable<'a>`. In this case the events we want to subscribe to are of type `ScoringEvent`.

The controller handles the events in a functional manner. Its signature `Score -> ScoringEvent -> Score` shows us that, whenever an event occurs, the current value of the model is transformed into a new value. This allows for our model to remain pure, although our view model is not.

An `eventHandler` is in charge of mutating the state of the view. Inheriting from `EventViewModelBase<'a>` we can use `EventValueCommand` and `EventValueCommandChecked` to hook up the events to the commands.

```fsharp
namespace Score.ViewModel

open Score.Model
open FSharp.ViewModule

type MainViewModel(controller : Score -> ScoringEvent -> Score) as self = 
    inherit EventViewModelBase<ScoringEvent>()

    let score = self.Factory.Backing(<@ self.Score @>, Score.zero)

    let eventHandler ev = score.Value <- controller score.Value ev

    do
        self.EventStream
        |> Observable.add eventHandler

    member this.IncA = this.Factory.EventValueCommand(IncA)
    member this.DecA = this.Factory.EventValueCommandChecked(DecA, (fun _ -> this.Score.ScoreA > 0), [ <@@ this.Score @@> ])
    member this.IncB = this.Factory.EventValueCommand(IncB)
    member this.DecB = this.Factory.EventValueCommandChecked(DecB, (fun _ -> this.Score.ScoreB > 0), [ <@@ this.Score @@> ])
    member this.NewGame = this.Factory.EventValueCommand(NewGame)

    member __.Score = score.Value
```

The code behind file (`*.xaml.fs`) is where everything is put together, i.e. the update function (`controller`) is injected in the `MainViewModel`.

```fsharp
namespace Score.Views

open FsXaml

type MainView = XAML<"MainWindow.xaml">

type CompositionRoot() =
    member __.ViewModel = Score.ViewModel.MainViewModel(Score.Model.Score.update)
```

The type `CompositionRoot` serves as a wrapper that’s referenced in the XAML file.

```xaml
<Window.Resources>
    <ResourceDictionary>
        <local:CompositionRoot x:Key="CompositionRoot"/>
    </ResourceDictionary>
</Window.Resources>
<Window.DataContext>
    <Binding Source="{StaticResource CompositionRoot}" Path="ViewModel" />
</Window.DataContext>
```

I won’t dive any deeper into the XAML file as it’s basic WPF stuff, the entire project can be found on [GitHub](https://github.com/marisks/evented_mvvm/tree/mvc_refactored).