---
title: Are we doing MVC wrong?
author: Liam McLennan
date: 2013-10-20 20:32
template: article.jade
---

The naive way to build user interfaces with the MVC pattern is to include the application's logic in controller actions - effectively recreating [transaction script](http://martinfowler.com/eaaCatalog/transactionScript.html) that is: 

> organizes business logic by procedures where each procedure handles a single request from the presentation.

Recognising that this is bad, many [smart people](http://lostechies.com/jimmybogard/2013/07/17/how-we-do-mvc-4-years-later/) have thought about [how to do mvc better](http://paulstovell.com/blog/clean-aspnet-mvc-controllers). A couple of things still bother me:

1. What is the responsibility of the controller? They always seem to combine UI decisions (choosing a view, reading the request, http status codes, http redirects) with application logic.

1. The application itself has no interface. Controllers are a UI that consume the application, but there is no clear application boundary that defines what the application itself does.

To solve these problems I have started using a different technique for mvc. I will demonstrate by refactoring a typical controller.

A Typical Controller
--------------------

    public class LibraryController : Controller
    {
        private readonly IBookRepository _bookRepository;
        private readonly IBorrowings _borrowings;
        private readonly IUsers _users;

        public LibraryController(IBookRepository bookRepository, IBorrowings borrowings, IUsers users)
        {
            _bookRepository = bookRepository;
            _borrowings = borrowings;
            _users = users;
        }

        public ActionResult Catalogue()
        {
            return View(_bookRepository.AllBooks());
        }

        public ActionResult Borrow(BookModel book)
        {
            var fullCatalogue = _bookRepository.AllBooks();
            var bookIsInCatalogue = fullCatalogue.Any(b => b.ISBN == book.ISBN);
            if (!bookIsInCatalogue)
            {
                SetErrorFlash("The selected book is not in the catalogue");
                return RedirectToAction("Catalogue");
            }

            var borrowedBooks = _borrowings.BorrowedBooks();
            var bookIsOutOnLoan = borrowedBooks.Any(b => b.ISBN == book.ISBN);
            if (bookIsOutOnLoan)
            {
                SetErrorFlash("The selected book is currently out on loan");
                return RedirectToAction("Catalogue");
            }

            var user = _users.GetAuthenticatedUser();
            var canBorrow = user.HasPermissionToBorrow();
            if (!canBorrow)
            {
                SetErrorFlash("The selected book is currently out on loan");
                return RedirectToAction("Catalogue");
            }

            var bookEntity = _bookRepository.GetByISBN(book.ISBN);
            var result = _borrowings.Borrow(bookEntity);

            if (result != null)
            {
                SetSuccessFlash("Book borrowed");
                return View("Borrowed", result);
            }
            else
            {
                return View("BorrowFailed");
            }
        }

        private void SetErrorFlash(string message) {}
        private void SetSuccessFlash(string message) {}
    }


One problem here is that there is no differentiation between the user interface behaviour and the function provided by the application. Both are implemented in the controller, mixed together in the same method (`Borrow`).

Another problem is that as the library functionality grows more dependencies will be added to the controller. Not all of the actions will use all of the dependencies, so cohesion is low, and coupling is high.

Application Feature API
-----------------------

Instead of the above controller I prefer to make the features an application provides into a first class API. I model each feature as a class, so the logic in `Borrow` action moves into a feature class called `BorrowABook`.

The genesis of the ideas show here come from Uncle Bob's presentation ['Architecture the Lost Years'](http://www.youtube.com/watch?v=WpkDN78P884).

    public class BorrowABook
    {
        public enum BorrowABookOutcomes
        {
            NotInCatalogue,
            OutOnLoan,
            InsufficientPermission,
            BorrowFailed,
            BorrowSucceeded
        }

        public Feature<BorrowABookOutcomes,Book> Borrow(BookModel book)
        {
            // .. implement application logic
            return new Feature<BorrowABookOutcomes,Book>(BorrowABookOutcomes.BorrowSucceeded, borrowedBook);
        }
    }

Features return a glorified tuple (`Feature<TOutcome,TResult>`) that combines an enum value that represents what the outcome was, with a value that represents the result of the operation. In this case I am returning the success outcome and the book that was borrowed. As the boundary of your application Features are an excellent target for behaviour tests. You can [test them without dealing with UI layer concerns](https://gist.github.com/liammclennan/78ea542e2fc120bc643b).  

Simplified Controller
---------------------

Given a feature such as `BorrowABook` the controller action simplifies to the following:

    public ActionResult Borrow(BookModel book)
    {
        var result = _feature.Borrow(book);
        switch (result.Outcome)
        {
            case BorrowABook.BorrowABookOutcomes.NotInCatalogue:
                {
                    SetErrorFlash("The selected book is not in the catalogue");
                    return RedirectToAction("Catalogue");
                }
            case BorrowABook.BorrowABookOutcomes.OutOnLoan:
                {
                    SetErrorFlash("The selected book is currently out on loan");
                    return RedirectToAction("Catalogue");
                }
            case BorrowABook.BorrowABookOutcomes.InsufficientPermission:
                {
                    SetErrorFlash("You do not have permission to borrow books");
                    return RedirectToAction("Catalogue");
                }
            case BorrowABook.BorrowABookOutcomes.BorrowFailed:
                {
                    return View("BorrowFailed");
                }
            case BorrowABook.BorrowABookOutcomes.BorrowSucceeded:
                {
                    SetSuccessFlash("Book borrowed");
                    return View("Borrowed", result.Result);
                }
            default: throw new Exception("Unknown result " + result.Outcome);
        }
    }

Note that the application logic has all been removed. The controller action is reduced to the minimum required to implement the UI.

Update
-----

Thanks to David Tchepak for providing [this F# version](https://gist.github.com/dtchepak/7168143) of the above example. Note that it is untested and meant simply as an example of what an F# version might look like.

<script src="https://gist.github.com/dtchepak/7168143.js"></script>
