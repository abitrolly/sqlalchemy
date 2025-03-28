=============
1.4 Changelog
=============

This document details individual issue-level changes made throughout
1.4 releases.  For a narrative overview of what's new in 1.4, see
:ref:`migration_14_toplevel`.


.. changelog_imports::

    .. include:: changelog_13.rst
        :start-line: 5


.. changelog::
    :version: 1.4.18
    :include_notes_from: unreleased_14

.. changelog::
    :version: 1.4.17
    :released: May 29, 2021

    .. change::
        :tags: bug, orm, regression
        :tickets: 6558

        Fixed regression caused by just-released performance fix mentioned in #6550
        where a query.join() to a relationship could produce an AttributeError if
        the query were made against non-ORM structures only, a fairly unusual
        calling pattern.

.. changelog::
    :version: 1.4.16
    :released: May 28, 2021

    .. change::
        :tags: bug, engine
        :tickets: 6482

        Fixed issue where an ``@`` sign in the database portion of a URL would not
        be interpreted correctly if the URL also had a username:password section.


    .. change::
        :tags: bug, ext
        :tickets: 6529

        Fixed a deprecation warning that was emitted when using
        :func:`_automap.automap_base` without passing an existing
        ``Base``.


    .. change::
        :tags: bug, pep484
        :tickets: 6461

        Remove pep484 types from the code.
        Current effort is around the stub package, and having typing in
        two places makes thing worse, since the types in the SQLAlchemy
        source were usually outdated compared to the version in the stubs.

    .. change::
        :tags: usecase, mssql
        :tickets: 6464

        Implemented support for a :class:`_sql.CTE` construct to be used directly
        as the target of a :func:`_sql.delete` construct, i.e. "WITH ... AS cte
        DELETE FROM cte". This appears to be a useful feature of SQL Server.

    .. change::
        :tags: bug, general
        :tickets: 6540, 6543

        Resolved various deprecation warnings which were appearing as of Python
        version 3.10.0b1.

    .. change::
        :tags: bug, orm
        :tickets: 6471

        Fixed issue when using :paramref:`_orm.relationship.cascade_backrefs`
        parameter set to ``False``, which per :ref:`change_5150` is set to become
        the standard behavior in SQLAlchemy 2.0, where adding the item to a
        collection that uniquifies, such as ``set`` or ``dict`` would fail to fire
        a cascade event if the object were already associated in that collection
        via the backref. This fix represents a fundamental change in the collection
        mechanics by introducing a new event state which can fire off for a
        collection mutation even if there is no net change on the collection; the
        action is now suited using a new event hook
        :meth:`_orm.AttributeEvents.append_wo_mutation`.



    .. change::
        :tags: bug, orm, regression
        :tickets: 6550

        Fixed regression involving clause adaption of labeled ORM compound
        elements, such as single-table inheritance discriminator expressions with
        conditionals or CASE expressions, which could cause aliased expressions
        such as those used in ORM join / joinedload operations to not be adapted
        correctly, such as referring to the wrong table in the ON clause in a join.

        This change also improves a performance bump that was located within the
        process of invoking :meth:`_sql.Select.join` given an ORM attribute
        as a target.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6495

        Fixed regression where the full combination of joined inheritance, global
        with_polymorphic, self-referential relationship and joined loading would
        fail to be able to produce a query with the scope of lazy loads and object
        refresh operations that also attempted to render the joined loader.

    .. change::
        :tags: bug, engine
        :tickets: 6329

        Fixed a long-standing issue with :class:`.URL` where query parameters
        following the question mark would not be parsed correctly if the URL did
        not contain a database portion with a backslash.

    .. change::
        :tags: bug, sql, regression
        :tickets: 6549

        Fixed regression in dynamic loader strategy and :func:`_orm.relationship`
        overall where the :paramref:`_orm.relationship.order_by` parameter were
        stored as a mutable list, which could then be mutated when combined with
        additional "order_by" methods used against the dynamic query object,
        causing the ORDER BY criteria to continue to grow repetitively.

    .. change::
        :tags: bug, orm
        :tickets: 6484

        Enhanced the bind resolution rules for :meth:`_orm.Session.execute` so that
        when a non-ORM statement such as an :func:`_sql.insert` construct
        nonetheless is built against ORM objects, to the greatest degree possible
        the ORM entity will be used to resolve the bind, such as for a
        :class:`_orm.Session` that has a bind map set up on a common superclass
        without specific mappers or tables named in the map.

    .. change::
        :tags: bug, regression, ext
        :tickets: 6390

        Fixed regression in the ``sqlalchemy.ext.instrumentation`` extension that
        prevented instrumentation disposal from working completely. This fix
        includes both a 1.4 regression fix as well as a fix for a related issue
        that existed in 1.3 also.   As part of this change, the
        :class:`sqlalchemy.ext.instrumentation.InstrumentationManager` class now
        has a new method ``unregister()``, which replaces the previous method
        ``dispose()``, which was not called as of version 1.4.


.. changelog::
    :version: 1.4.15
    :released: May 11, 2021

    .. change::
        :tags: bug, documentation, mysql
        :tickets: 5397

        Added support for the ``ssl_check_hostname=`` parameter in mysql connection
        URIs and updated the mysql dialect documentation regarding secure
        connections. Original pull request courtesy of Jerry Zhao.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6449

        Fixed additional regression caused by "eager loaders run on unexpire"
        feature :ticket:`1763` where the feature would run for a
        ``contains_eager()`` eagerload option in the case that the
        ``contains_eager()`` were chained to an additional eager loader option,
        which would then produce an incorrect query as the original query-bound
        join criteria were no longer present.

    .. change::
        :tags: feature, general
        :tickets: 6241

        A new approach has been applied to the warnings system in SQLAlchemy to
        accurately predict the appropriate stack level for each warning
        dynamically. This allows evaluating the source of SQLAlchemy-generated
        warnings and deprecation warnings to be more straightforward as the warning
        will indicate the source line within end-user code, rather than from an
        arbitrary level within SQLAlchemy's own source code.

    .. change::
        :tags: bug, orm
        :tickets: 6459

        Fixed issue in subquery loader strategy which prevented caching from
        working correctly. This would have been seen in the logs as a "generated"
        message instead of "cached" for all subqueryload SQL emitted, which by
        saturating the cache with new keys would degrade overall performance; it
        also would produce "LRU size alert" warnings.


    .. change::
        :tags: bug, sql
        :tickets: 6460

        Adjusted the logic added as part of :ticket:`6397` in 1.4.12 so that
        internal mutation of the :class:`.BindParameter` object occurs within the
        clause construction phase as it did before, rather than in the compilation
        phase. In the latter case, the mutation still produced side effects against
        the incoming construct and additionally could potentially interfere with
        other internal mutation routines.

.. changelog::
    :version: 1.4.14
    :released: May 6, 2021

    .. change::
        :tags: bug, regression, orm
        :tickets: 6426

        Fixed regression involving ``lazy='dynamic'`` loader in conjunction with a
        detached object. The previous behavior was that the dynamic loader upon
        calling methods like ``.all()`` returns empty lists for detached objects
        without error, this has been restored; however a warning is now emitted as
        this is not the correct result. Other dynamic loader scenarios correctly
        raise ``DetachedInstanceError``.

    .. change::
        :tags: bug, regression, sql
        :tickets: 6428

        Fixed regression caused by the "empty in" change just made in
        :ticket:`6397` 1.4.12 where the expression needs to be parenthesized for
        the "not in" use case, otherwise the condition will interfere with the
        other filtering criteria.


    .. change::
        :tags: bug, sql, regression
        :tickets: 6436

        The :class:`.TypeDecorator` class will now emit a warning when used in SQL
        compilation with caching unless the ``.cache_ok`` flag is set to ``True``
        or ``False``. A new class-level attribute :attr:`.TypeDecorator.cache_ok`
        may be set which will be used as an indication that all the parameters
        passed to the object are safe to be used as a cache key if set to ``True``,
        ``False`` means they are not.

    .. change::
        :tags: engine, bug, regression
        :tickets: 6427

        Established a deprecation path for calling upon the
        :meth:`_cursor.CursorResult.keys` method for a statement that returns no
        rows to provide support for legacy patterns used by the "records" package
        as well as any other non-migrated applications. Previously, this would
        raise :class:`.ResourceClosedException` unconditionally in the same way as
        it does when attempting to fetch rows. While this is the correct behavior
        going forward, the :class:`_cursor.LegacyCursorResult` object will now in
        this case return an empty list for ``.keys()`` as it did in 1.3, while also
        emitting a 2.0 deprecation warning. The :class:`_cursor.CursorResult`, used
        when using a 2.0-style "future" engine, will continue to raise as it does
        now.

    .. change::
        :tags: usecase, engine, orm
        :tickets: 6288

        Applied consistent behavior to the use case of
        calling ``.commit()`` or ``.rollback()`` inside of an existing
        ``.begin()`` context manager, with the addition of potentially
        emitting SQL within the block subsequent to the commit or rollback.
        This change continues upon the change first added in
        :ticket:`6155` where the use case of calling "rollback" inside of
        a ``.begin()`` contextmanager block was proposed:

        * calling ``.commit()`` or ``.rollback()`` will now be allowed
          without error or warning within all scopes, including
          that of legacy and future :class:`_engine.Engine`, ORM
          :class:`_orm.Session`, asyncio :class:`.AsyncEngine`.  Previously,
          the :class:`_orm.Session` disallowed this.

        * The remaining scope of the context manager is then closed;
          when the block ends, a check is emitted to see if the transaction
          was already ended, and if so the block returns without action.

        * It will now raise **an error** if subsequent SQL of any kind
          is emitted within the block, **after** ``.commit()`` or
          ``.rollback()`` is called.   The block should be closed as
          the state of the executable object would otherwise be undefined
          in this state.

.. changelog::
    :version: 1.4.13
    :released: May 3, 2021

    .. change::
        :tags: bug, regression, orm
        :tickets: 6410

        Fixed regression in ``selectinload`` loader strategy that would cause it to
        cache its internal state incorrectly when handling relationships that join
        across more than one column, such as when using a composite foreign key.
        The invalid caching would then cause other unrelated loader operations to
        fail.


    .. change::
        :tags: bug, orm, regression
        :tickets: 6414

        Fixed regression where :meth:`_orm.Query.filter_by` would not work if the
        lead entity were a SQL function or other expression derived from the
        primary entity in question, rather than a simple entity or column of that
        entity. Additionally, improved the behavior of
        :meth:`_sql.Select.filter_by` overall to work with column expressions even
        in a non-ORM context.

    .. change::
        :tags: bug, engine, regression
        :tickets: 6408

        Restored a legacy transactional behavior that was inadvertently removed
        from the :class:`_engine.Connection` as it was never tested as a known use
        case in previous versions, where calling upon the
        :meth:`_engine.Connection.begin_nested` method, when no transaction is
        present, does not create a SAVEPOINT at all and instead starts an outer
        transaction, returning a :class:`.RootTransaction` object instead of a
        :class:`.NestedTransaction` object.  This :class:`.RootTransaction` then
        will emit a real COMMIT on the database connection when committed.
        Previously, the 2.0 style behavior was present in all cases that would
        autobegin a transaction but not commit it, which is a behavioral change.

        When using a :term:`2.0 style` connection object, the behavior is unchanged
        from previous 1.4 versions; calling :meth:`_future.Connection.begin_nested`
        will "autobegin" the outer transaction if not already present, and then as
        instructed emit a SAVEPOINT, returning the :class:`.NestedTransaction`
        object. The outer transaction is committed by calling upon
        :meth:`_future.Connection.commit`, as is "commit-as-you-go" style usage.

        In non-"future" mode, while the old behavior is restored, it also
        emits a 2.0 deprecation warning as this is a legacy behavior.


    .. change::
        :tags: bug, asyncio, regression
        :tickets: 6409

        Fixed a regression introduced by :ticket:`6337` that would create an
        ``asyncio.Lock`` which could be attached to the wrong loop when
        instantiating the async engine before any asyncio loop was started, leading
        to an asyncio error message when attempting to use the engine under certain
        circumstances.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6419

        Fixed regression where using :func:`_orm.selectinload` and
        :func:`_orm.subqueryload` to load a two-level-deep path would lead to an
        attribute error.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6420

        Fixed regression where using the :func:`_orm.noload` loader strategy in
        conjunction with a "dynamic" relationship would lead to an attribute error
        as the noload strategy would attempt to apply itself to the dynamic loader.

    .. change::
        :tags: usecase, postgresql
        :tickets: 6198

        Add support for server side cursors in the pg8000 dialect for PostgreSQL.
        This allows use of the
        :paramref:`.Connection.execution_options.stream_results` option.

.. changelog::
    :version: 1.4.12
    :released: April 29, 2021

    .. change::
        :tags: bug, orm, regression, caching
        :tickets: 6391

        Fixed critical regression where bound parameter tracking as used in the SQL
        caching system could fail to track all parameters for the case where the
        same SQL expression containing a parameter were used in an ORM-related
        query using a feature such as class inheritance, which was then embedded in
        an enclosing expression which would make use of that same expression
        multiple times, such as a UNION. The ORM would individually copy the
        individual SELECT statements as part of compilation with class inheritance,
        which then embedded in the enclosing statement would fail to accommodate
        for all parameters. The logic that tracks this condition has been adjusted
        to work for multiple copies of a parameter.

    .. change::
        :tags: bug, sql
        :tickets: 6258, 6397

        Revised the "EMPTY IN" expression to no longer rely upon using a subquery,
        as this was causing some compatibility and performance problems. The new
        approach for selected databases takes advantage of using a NULL-returning
        IN expression combined with the usual "1 != 1" or "1 = 1" expression
        appended by AND or OR. The expression is now the default for all backends
        other than SQLite, which still had some compatibility issues regarding
        tuple "IN" for older SQLite versions.

        Third party dialects can still override how the "empty set" expression
        renders by implementing a new compiler method
        ``def visit_empty_set_op_expr(self, type_, expand_op)``, which takes
        precedence over the existing
        ``def visit_empty_set_expr(self, element_types)`` which remains in place.


    .. change::
        :tags: bug, orm
        :tickets: 6350

        Fixed two distinct issues mostly affecting
        :class:`_hybrid.hybrid_property`, which would come into play under common
        mis-configuration scenarios that were silently ignored in 1.3, and now
        failed in 1.4, where the "expression" implementation would return a non
        :class:`_sql.ClauseElement` such as a boolean value. For both issues, 1.3's
        behavior was to silently ignore the mis-configuration and ultimately
        attempt to interpret the value as a SQL expression, which would lead to an
        incorrect query.

        * Fixed issue regarding interaction of the attribute system with
          hybrid_property, where if the ``__clause_element__()`` method of the
          attribute returned a non-:class:`_sql.ClauseElement` object, an internal
          ``AttributeError`` would lead the attribute to return the ``expression``
          function on the hybrid_property itself, as the attribute error was
          against the name ``.expression`` which would invoke the ``__getattr__()``
          method as a fallback. This now raises explicitly. In 1.3 the
          non-:class:`_sql.ClauseElement` was returned directly.

        * Fixed issue in SQL argument coercions system where passing the wrong
          kind of object to methods that expect column expressions would fail if
          the object were altogether not a SQLAlchemy object, such as a Python
          function, in cases where the object were not just coerced into a bound
          value. Again 1.3 did not have a comprehensive argument coercion system
          so this case would also pass silently.


    .. change::
        :tags: bug, orm
        :tickets: 6378

        Fixed issue where using a :class:`_sql.Select` as a subquery in an ORM
        context would modify the :class:`_sql.Select` in place to disable
        eagerloads on that object, which would then cause that same
        :class:`_sql.Select` to not eagerload if it were then re-used in a
        top-level execution context.


    .. change::
        :tags: bug, regression, sql
        :tickets: 6343

        Fixed regression where usage of the :func:`_sql.text` construct inside the
        columns clause of a :class:`_sql.Select` construct, which is better handled
        by using a :func:`_sql.literal_column` construct, would nonetheless prevent
        constructs like :func:`_sql.union` from working correctly. Other use cases,
        such as constructing subuqeries, continue to work the same as in prior
        versions where the :func:`_sql.text` construct is silently omitted from the
        collection of exported columns.   Also repairs similar use within the
        ORM.


    .. change::
        :tags: bug, regression, sql
        :tickets: 6261

        Fixed regression involving legacy methods such as
        :meth:`_sql.Select.append_column` where internal assertions would fail.

    .. change::
        :tags: usecase, sqlite
        :tickets: 6379

        Default to using ``SingletonThreadPool`` for in-memory SQLite databases
        created using URI filenames. Previously the default pool used was the
        ``NullPool`` that precented sharing the same database between multiple
        engines.

    .. change::
        :tags: bug, regression, sql
        :tickets: 6300

        Fixed regression caused by :ticket:`5395` where tuning back the check for
        sequences in :func:`_sql.select` now caused failures when doing 2.0-style
        querying with a mapped class that also happens to have an ``__iter__()``
        method. Tuned the check some more to accommodate this as well as some other
        interesting ``__iter__()`` scenarios.


    .. change::
        :tags: bug, mssql, schema
        :tickets: 6345

        Add :meth:`_types.TypeEngine.as_generic` support for
        :class:`sqlalchemy.dialects.mysql.BIT` columns, mapping
        them to :class:`_sql.sqltypes.Boolean`.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6360, 6359

        Fixed issue where the new :ref:`autobegin <session_autobegin>` behavior
        failed to "autobegin" in the case where an existing persistent object has
        an attribute change, which would then impact the behavior of
        :meth:`_orm.Session.rollback` in that no snapshot was created to be rolled
        back. The "attribute modify" mechanics have been updated to ensure
        "autobegin", which does not perform any database work, does occur when
        persistent attributes change in the same manner as when
        :meth:`_orm.Session.add` is called. This is a regression as in 1.3, the
        rollback() method always had a transaction to roll back and would expire
        every time.

    .. change::
        :tags: bug, mssql, regression
        :tickets: 6366

        Fixed regression caused by :ticket:`6306` which added support for
        ``DateTime(timezone=True)``, where the previous behavior of the pyodbc
        driver of implicitly dropping the tzinfo from a timezone-aware date when
        INSERTing into a timezone-naive DATETIME column were lost, leading to a SQL
        Server error when inserting timezone-aware datetime objects into
        timezone-native database columns.

    .. change::
        :tags: orm, bug, regression
        :tickets: 6386

        Fixed regression in ORM where using hybrid property to indicate an
        expression from a different entity would confuse the column-labeling logic
        in the ORM and attempt to derive the name of the hybrid from that other
        class, leading to an attribute error. The owning class of the hybrid
        attribute is now tracked along with the name.

    .. change::
        :tags: orm, bug, regression
        :tickets: 6401

        Fixed regression in hybrid_property where a hybrid against a SQL function
        would generate an ``AttributeError`` when attempting to generate an entry
        for the ``.c`` collection of a subquery in some cases; among other things
        this would impact its use in cases like that of ``Query.count()``.


    .. change::
        :tags: bug, postgresql
        :tickets: 6373

        Fixed very old issue where the :class:`_types.Enum` datatype would not
        inherit the :paramref:`_schema.MetaData.schema` parameter of a
        :class:`_schema.MetaData` object when that object were passed to the
        :class:`_types.Enum` using :paramref:`_types.Enum.metadata`.

    .. change::
        :tags: bug, orm, dataclasses
        :tickets: 6346

        Adjusted the declarative scan for dataclasses so that the inheritance
        behavior of :func:`_orm.declared_attr` established on a mixin, when using
        the new form of having it inside of a ``dataclasses.field()`` construct and
        not actually a descriptor attribute on the class, correctly accommodates
        the case when the target class to be mapped is a subclass of an existing
        mapped class which has already mapped that :func:`_orm.declared_attr`, and
        therefore should not be re-applied to this class.


    .. change::
        :tags: bug, schema, mysql, mariadb, oracle, postgresql
        :tickets: 6338

        Ensure that the MySQL and MariaDB dialect ignore the
        :class:`_sql.Identity` construct while rendering the ``AUTO_INCREMENT``
        keyword in a create table.

        The Oracle and PostgreSQL compiler was updated to not render
        :class:`_sql.Identity` if the database version does not support it
        (Oracle < 12 and PostgreSQL < 10). Previously it was rendered regardless
        of the database version.

    .. change::
        :tags: bug, orm
        :tickets: 6353

        Fixed an issue with the (deprecated in 1.4)
        :meth:`_schema.ForeignKeyConstraint.copy` method that caused an error when
        invoked with the ``schema`` argument.

    .. change::
        :tags: bug, engine
        :tickets: 6361

        Fixed issue where usage of an explicit :class:`.Sequence` would produce
        inconsistent "inline" behavior for an :class:`.Insert` construct that
        includes multiple values phrases; the first seq would be inline but
        subsequent ones would be "pre-execute", leading to inconsistent sequence
        ordering. The sequence expressions are now fully inline.

.. changelog::
    :version: 1.4.11
    :released: April 21, 2021

    .. change::
        :tags: bug, engine, regression
        :tickets: 6337

        Fixed critical regression caused by the change in :ticket:`5497` where the
        connection pool "init" phase no longer occurred within mutexed isolation,
        allowing other threads to proceed with the dialect uninitialized, which
        could then impact the compilation of SQL statements.


    .. change::
        :tags: bug, orm, regression, declarative
        :tickets: 6331

        Fixed regression where recent changes to support Python dataclasses had the
        inadvertent effect that an ORM mapped class could not successfully override
        the ``__new__()`` method.

.. changelog::
    :version: 1.4.10
    :released: April 20, 2021

    .. change::
        :tags: bug, declarative, regression
        :tickets: 6291

        Fixed :func:`_declarative.instrument_declarative` that called
        a non existing registry method.

    .. change::
        :tags: bug, orm
        :tickets: 6320

        Fixed bug in new :func:`_orm.with_loader_criteria` feature where using a
        mixin class with :func:`_orm.declared_attr` on an attribute that were
        accessed inside the custom lambda would emit a warning regarding using an
        unmapped declared attr, when the lambda callable were first initialized.
        This warning is now prevented using special instrumentation for this
        lambda initialization step.


    .. change::
        :tags: usecase, mssql
        :tickets: 6306

        The :paramref:`_types.DateTime.timezone` parameter when set to ``True``
        will now make use of the ``DATETIMEOFFSET`` column type with SQL Server
        when used to emit DDL, rather than ``DATETIME`` where the flag was silently
        ignored.

    .. change::
        :tags: orm, bug, regression
        :tickets: 6326

        Fixed additional regression caused by the "eagerloaders on refresh" feature
        added in :ticket:`1763` where the refresh operation historically would set
        ``populate_existing``, which given the new feature now overwrites pending
        changes on eagerly loaded objects when autoflush is false. The
        populate_existing flag has been turned off for this case and a more
        specific method used to ensure the correct attributes refreshed.

    .. change::
        :tags: bug, orm, result
        :tickets: 6299

        Fixed an issue when using 2.0 style execution that prevented using
        :meth:`_result.Result.scalar_one` or
        :meth:`_result.Result.scalar_one_or_none` after calling
        :meth:`_result.Result.unique`, for the case where the ORM is returning a
        single-element row in any case.

    .. change::
        :tags: bug, sql
        :tickets: 6327

        Fixed issue in SQL compiler where the bound parameters set up for a
        :class:`.Values` construct wouldn't be positionally tracked correctly if
        inside of a :class:`_sql.CTE`, affecting database drivers that support
        VALUES + ctes and use positional parameters such as SQL Server in
        particular as well as asyncpg.   The fix also repairs support for
        compiler flags such as ``literal_binds``.

    .. change::
        :tags: bug, schema
        :tickets: 6287

        Fixed issue where :func:`_functions.next_value` was not deriving its type
        from the corresponding :class:`_schema.Sequence`, instead hardcoded to
        :class:`_types.Integer`. The specific numeric type is now used.

    .. change::
        :tags: bug, mypy
        :tickets: 6255

        Fixed issue where mypy plugin would not correctly interpret an explicit
        :class:`_orm.Mapped` annotation in conjunction with a
        :func:`_orm.relationship` that refers to a class by string name; the
        correct annotation would be downgraded to a less specific one leading to
        typing errors.

    .. change::
        :tags: bug, sql
        :tickets: 6256

        Repaired and solidified issues regarding custom functions and other
        arbitrary expression constructs which within SQLAlchemy's column labeling
        mechanics would seek to use ``str(obj)`` to get a string representation to
        use as an anonymous column name in the ``.c`` collection of a subquery.
        This is a very legacy behavior that performs poorly and leads to lots of
        issues, so has been revised to no longer perform any compilation by
        establishing specific methods on :class:`.FunctionElement` to handle this
        case, as SQL functions are the only use case that it came into play. An
        effect of this behavior is that an unlabeled column expression with no
        derivable name will be given an arbitrary label starting with the prefix
        ``"_no_label"`` in the ``.c`` collection of a subquery; these were
        previously being represented either as the generic stringification of that
        expression, or as an internal symbol.

    .. change::
        :tags: usecase, orm
        :ticketS: 6301

        Altered some of the behavior repaired in :ticket:`6232` where the
        ``immediateload`` loader strategy no longer goes into recursive loops; the
        modification is that an eager load (joinedload, selectinload, or
        subqueryload) from A->bs->B which then states ``immediateload`` for a
        simple manytoone B->a->A that's in the identity map will populate the B->A,
        so that this attribute is back-populated when the collection of A/A.bs are
        loaded. This allows the objects to be functional when detached.


.. changelog::
    :version: 1.4.9
    :released: April 17, 2021

    .. change::
        :tags: bug, sql, regression
        :tickets: 6290

        Fixed regression where an empty in statement on a tuple would result
        in an error when compiled with the option ``literal_binds=True``.

    .. change::
        :tags: bug, regression, orm, performance, sql
        :tickets: 6304

        Fixed a critical performance issue where the traversal of a
        :func:`_sql.select` construct would traverse a repetitive product of the
        represented FROM clauses as they were each referred towards by columns in
        the columns clause; for a series of nested subqueries with lots of columns
        this could cause a large delay and significant memory growth. This
        traversal is used by a wide variety of SQL and ORM functions, including by
        the ORM :class:`_orm.Session` when it's configured to have
        "table-per-bind", which while this is not a common use case, it seems to be
        what Flask-SQLAlchemy is hardcoded as using, so the issue impacts
        Flask-SQLAlchemy users. The traversal has been repaired to uniqify on FROM
        clauses which was effectively what would happen implicitly with the pre-1.4
        architecture.

    .. change::
        :tags: bug, postgresql, sql, regression
        :tickets: 6303

        Fixed an argument error in the default and PostgreSQL compilers that
        would interfere with an UPDATE..FROM or DELETE..FROM..USING statement
        that was then SELECTed from as a CTE.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6272

        Fixed regression where an attribute that is mapped to a
        :func:`_orm.synonym` could not be used in column loader options such as
        :func:`_orm.load_only`.

    .. change::
        :tags: usecase, orm
        :tickets: 6267

        Established support for :func:`_orm.synoynm` in conjunction with
        hybrid property, assocaitionproxy is set up completely, including that
        synonyms can be established linking to these constructs which work
        fully.   This is a behavior that was semi-explicitly disallowed previously,
        however since it did not fail in every scenario, explicit support
        for assoc proxy and hybrids has been added.


.. changelog::
    :version: 1.4.8
    :released: April 15, 2021

    .. change::
        :tags: change, mypy

        Updated Mypy plugin to only use the public plugin interface of the
        semantic analyzer.

    .. change::
        :tags: bug, mssql, regression
        :tickets: 6265

        Fixed an additional regression in the same area as that of :ticket:`6173`,
        :ticket:`6184`, where using a value of 0 for OFFSET in conjunction with
        LIMIT with SQL Server would create a statement using "TOP", as was the
        behavior in 1.3, however due to caching would then fail to respond
        accordingly to other values of OFFSET. If the "0" wasn't first, then it
        would be fine. For the fix, the "TOP" syntax is now only emitted if the
        OFFSET value is omitted entirely, that is, :meth:`_sql.Select.offset` is
        not used. Note that this change now requires that if the "with_ties" or
        "percent" modifiers are used, the statement can't specify an OFFSET of
        zero, it now needs to be omitted entirely.

    .. change::
        :tags: bug, engine

        The :meth:`_engine.Dialect.has_table` method now raises an informative
        exception if a non-Connection is passed to it, as this incorrect behavior
        seems to be common.  This method is not intended for external use outside
        of a dialect.  Please use the :meth:`.Inspector.has_table` method
        or for cross-compatibility with older SQLAlchemy versions, the
        :meth:`_engine.Engine.has_table` method.


    .. change::
        :tags: bug, regression, sql
        :tickets: 6249

        Fixed regression where the :class:`_sql.BindParameter` object would not
        properly render for an IN expression (i.e. using the "post compile" feature
        in 1.4) if the object were copied from either an internal cloning
        operation, or from a pickle operation, and the parameter name contained
        spaces or other special characters.

    .. change::
        :tags: bug, mypy
        :tickets: 6205

        Revised the fix for ``OrderingList`` from version 1.4.7 which was testing
        against the incorrect API.

    .. change::
        :tags: bug, asyncio
        :tickets: 6220

        Fix typo that prevented setting the ``bind`` attribute of an
        :class:`_asyncio.AsyncSession` to the correct value.

    .. change::
        :tags: feature, sql
        :tickets: 3314

        The tuple returned by :attr:`.CursorResult.inserted_primary_key` is now a
        :class:`_result.Row` object with a named tuple interface on top of the
        existing tuple interface.




    .. change::
        :tags: bug, regression, sql, sqlite
        :tickets: 6254

        Fixed regression where the introduction of the INSERT syntax "INSERT...
        VALUES (DEFAULT)" was not supported on some backends that do however
        support "INSERT..DEFAULT VALUES", including SQLite. The two syntaxes are
        now each individually supported or non-supported for each dialect, for
        example MySQL supports "VALUES (DEFAULT)" but not "DEFAULT VALUES".
        Support for Oracle has also been enabled.

    .. change::
        :tags: bug, regression, orm
        :tickets: 6259

        Fixed a cache leak involving the :func:`_orm.with_expression` loader
        option, where the given SQL expression would not be correctly considered as
        part of the cache key.

        Additionally, fixed regression involving the corresponding
        :func:`_orm.query_expression` feature. While the bug technically exists in
        1.3 as well, it was not exposed until 1.4. The "default expr" value of
        ``null()`` would be rendered when not needed, and additionally was also not
        adapted correctly when the ORM rewrites statements such as when using
        joined eager loading. The fix ensures "singleton" expressions like ``NULL``
        and ``true`` aren't "adapted" to refer to columns in ORM statements, and
        additionally ensures that a :func:`_orm.query_expression` with no default
        expression doesn't render in the statement if a
        :func:`_orm.with_expression` isn't used.

    .. change::
        :tags: bug, orm
        :tickets: 6252

        Fixed issue in the new feature of :meth:`_orm.Session.refresh` introduced
        by :ticket:`1763` where eagerly loaded relationships are also refreshed,
        where the ``lazy="raise"`` and ``lazy="raise_on_sql"`` loader strategies
        would interfere with the :func:`_orm.immediateload` loader strategy, thus
        breaking the feature for relationships that were loaded with
        :func:`_orm.selectinload`, :func:`_orm.subqueryload` as well.

.. changelog::
    :version: 1.4.7
    :released: April 9, 2021

    .. change::
        :tags: bug, sql, regression
        :tickets: 6222

        Enhanced the "expanding" feature used for :meth:`_sql.ColumnOperators.in_`
        operations to infer the type of expression from the right hand list of
        elements, if the left hand side does not have any explicit type set up.
        This allows the expression to support stringification among other things.
        In 1.3, "expanding" was not automatically used for
        :meth:`_sql.ColumnOperators.in_` expressions, so in that sense this change
        fixes a behavioral regression.


    .. change::
        :tags: bug, mypy

        Fixed issue in Mypy plugin where the plugin wasn’t inferring the correct
        type for columns of subclasses that don’t directly descend from
        ``TypeEngine``, in particular that of  ``TypeDecorator`` and
        ``UserDefinedType``.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6221

        Fixed regression where the :func:`_orm.subqueryload` loader strategy would
        fail to correctly accommodate sub-options, such as a :func:`_orm.defer`
        option on a column, if the "path" of the subqueryload were more than one
        level deep.


    .. change::
        :tags: bug, sql

        Fixed the "stringify" compiler to support a basic stringification
        of a "multirow" INSERT statement, i.e. one with multiple tuples
        following the VALUES keyword.


    .. change::
        :tags: bug, orm, regression
        :tickets: 6211

        Fixed regression where the :func:`_orm.merge_frozen_result` function relied
        upon by the dogpile.caching example was not included in tests and began
        failing due to incorrect internal arguments.

    .. change::
        :tags: bug, engine, regression
        :tickets: 6218

        Fixed up the behavior of the :class:`_result.Row` object when dictionary
        access is used upon it, meaning converting to a dict via ``dict(row)`` or
        accessing members using strings or other objects i.e. ``row["some_key"]``
        works as it would with a dictionary, rather than raising ``TypeError`` as
        would be the case with a tuple, whether or not the C extensions are in
        place. This was originally supposed to emit a 2.0 deprecation warning for
        the "non-future" case using :class:`_result.LegacyRow`, and was to raise
        ``TypeError`` for the "future" :class:`_result.Row` class. However, the C
        version of :class:`_result.Row` was failing to raise this ``TypeError``,
        and to complicate matters, the :meth:`_orm.Session.execute` method now
        returns :class:`_result.Row` in all cases to maintain consistency with the
        ORM result case, so users who didn't have C extensions installed would
        see different behavior in this one case for existing pre-1.4 style
        code.

        Therefore, in order to soften the overall upgrade scheme as most users have
        not been exposed to the more strict behavior of :class:`_result.Row` up
        through 1.4.6, :class:`_result.LegacyRow` and :class:`_result.Row` both
        provide for string-key access as well as support for ``dict(row)``, in all
        cases emitting the 2.0 deprecation warning when ``SQLALCHEMY_WARN_20`` is
        enabled. The :class:`_result.Row` object still uses tuple-like behavior for
        ``__contains__``, which is probably the only noticeable behavioral change
        compared to :class:`_result.LegacyRow`, other than the removal of
        dictionary-style methods ``values()`` and ``items()``.

    .. change::
        :tags: bug, regression, orm
        :tickets: 6233

        Fixed critical regression where the :class:`_orm.Session` could fail to
        "autobegin" a new transaction when a flush occurred without an existing
        transaction in place, implicitly placing the :class:`_orm.Session` into
        legacy autocommit mode which commit the transaction. The
        :class:`_orm.Session` now has a check that will prevent this condition from
        occurring, in addition to repairing the flush issue.

        Additionally, scaled back part of the change made as part of :ticket:`5226`
        which can run autoflush during an unexpire operation, to not actually
        do this in the case of a :class:`_orm.Session` using legacy
        :paramref:`_orm.Session.autocommit` mode, as this incurs a commit within
        a refresh operation.

    .. change::
        :tags: change, tests

        Added a new flag to :class:`.DefaultDialect` called ``supports_schemas``;
        third party dialects may set this flag to ``False`` to disable SQLAlchemy's
        schema-level tests when running the test suite for a third party dialect.

    .. change::
        :tags: bug, regression, schema
        :tickets: 6216

        Fixed regression where usage of a token in the
        :paramref:`_engine.Connection.execution_options.schema_translate_map`
        dictionary which contained special characters such as braces would fail to
        be substituted properly. Use of square bracket characters ``[]`` is now
        explicitly disallowed as these are used as a delimiter character in the
        current implementation.

    .. change::
        :tags: bug, regression, orm
        :tickets: 6215

        Fixed regression where the ORM compilation scheme would assume the function
        name of a hybrid property would be the same as the attribute name in such a
        way that an ``AttributeError`` would be raised, when it would attempt to
        determine the correct name for each element in a result tuple. A similar
        issue exists in 1.3 but only impacts the names of tuple rows. The fix here
        adds a check that the hybrid's function name is actually present in the
        ``__dict__`` of the class or its superclasses before assigning this name;
        otherwise, the hybrid is considered to be "unnamed" and ORM result tuples
        will use the naming scheme of the underlying expression.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6232

        Fixed critical regression caused by the new feature added as part of
        :ticket:`1763`, eager loaders are invoked on unexpire operations. The new
        feature makes use of the "immediateload" eager loader strategy as a
        substitute for a collection loading strategy, which unlike the other
        "post-load" strategies was not accommodating for recursive invocations
        between mutually-dependent relationships, leading to recursion overflow
        errors.


.. changelog::
    :version: 1.4.6
    :released: April 6, 2021

    .. change::
        :tags: bug, sql, regression, oracle, mssql
        :tickets: 6202

        Fixed further regressions in the same area as that of :ticket:`6173` released in
        1.4.5, where a "postcompile" parameter, again most typically those used for
        LIMIT/OFFSET rendering in Oracle and SQL Server, would fail to be processed
        correctly if the same parameter rendered in multiple places in the
        statement.



    .. change::
        :tags: bug, orm, regression
        :tickets: 6203

        Fixed regression where a deprecated form of :meth:`_orm.Query.join` were
        used, passing a series of entities to join from without any ON clause in a
        single :meth:`_orm.Query.join` call, would fail to function correctly.

    .. change::
        :tags: bug, mypy
        :tickets: 6147

        Applied a series of refactorings and fixes to accommodate for Mypy
        "incremental" mode across multiple files, which previously was not taken
        into account. In this mode the Mypy plugin has to accommodate Python
        datatypes expressed in other files coming in with less information than
        they have on a direct run.

        Additionally, a new decorator :func:`_orm.declarative_mixin` is added,
        which is necessary for the Mypy plugin to be able to definifitely identify
        a Declarative mixin class that is otherwise not used inside a particular
        Python file.

        .. seealso::

            :ref:`mypy_declarative_mixins`


    .. change::
        :tags: bug, mypy
        :tickets: 6205

        Fixed issue where the Mypy plugin would fail to interpret the
        "collection_class" of a relationship if it were a callable and not a class.
        Also improved type matching and error reporting for collection-oriented
        relationships.


    .. change::
        :tags: bug, sql
        :tickets: 6204

        Executing a :class:`_sql.Subquery` using :meth:`_engine.Connection.execute`
        is deprecated and will emit a deprecation warning; this use case was an
        oversight that should have been removed from 1.4. The operation will now
        execute the underlying :class:`_sql.Select` object directly for backwards
        compatibility. Similarly, the :class:`_sql.CTE` class is also not
        appropriate for execution. In 1.3, attempting to execute a CTE would result
        in an invalid "blank" SQL statement being executed; since this use case was
        not working it now raises :class:`_exc.ObjectNotExecutableError`.
        Previously, 1.4 was attempting to execute the CTE as a statement however it
        was working only erratically.

    .. change::
        :tags: bug, regression, orm
        :tickets: 6206

        Fixed critical regression where the :meth:`_orm.Query.yield_per` method in
        the ORM would set up the internal :class:`_engine.Result` to yield chunks
        at a time, however made use of the new :meth:`_engine.Result.unique` method
        which uniques across the entire result. This would lead to lost rows since
        the ORM is using ``id(obj)`` as the uniquing function, which leads to
        repeated identifiers for new objects as already-seen objects are garbage
        collected. 1.3's behavior here was to "unique" across each chunk, which
        does not actually produce "uniqued" results when results are yielded in
        chunks. As the :meth:`_orm.Query.yield_per` method is already explicitly
        disallowed when joined eager loading is in place, which is the primary
        rationale for the "uniquing" feature, the "uniquing" feature is now turned
        off entirely when :meth:`_orm.Query.yield_per` is used.

        This regression only applies to the legacy :class:`_orm.Query` object; when
        using :term:`2.0 style` execution, "uniquing" is not automatically applied.
        To prevent the issue from arising from explicit use of
        :meth:`_engine.Result.unique`, an error is now raised if rows are fetched
        from a "uniqued" ORM-level :class:`_engine.Result` if any
        :ref:`yield per <orm_queryguide_yield_per>` API is also in use, as the
        purpose of ``yield_per`` is to allow for arbitrarily large numbers of rows,
        which cannot be uniqued in memory without growing the number of entries to
        fit the complete result size.


    .. change::
        :tags: usecase, asyncio, postgresql
        :tickets: 6199

        Added accessors ``.sqlstate`` and synonym ``.pgcode`` to the ``.orig``
        attribute of the SQLAlchemy exception class raised by the asyncpg DBAPI
        adapter, that is, the intermediary exception object that wraps on top of
        that raised by the asyncpg library itself, but below the level of the
        SQLAlchemy dialect.

.. changelog::
    :version: 1.4.5
    :released: April 2, 2021

    .. change::
        :tags: bug, sql, postgresql
        :tickets: 6183

        Fixed bug in new :meth:`_functions.FunctionElement.render_derived` feature
        where column names rendered out explicitly in the alias SQL would not have
        proper quoting applied for case sensitive names and other non-alphanumeric
        names.

    .. change::
        :tags: bug, regression, orm
        :tickets: 6172

        Fixed regression where the :func:`_orm.joinedload` loader strategy would
        not successfully joinedload to a mapper that is mapper against a
        :class:`.CTE` construct.

    .. change::
        :tags: bug, regression, sql
        :tickets: 6181

        Fixed regression where use of the :meth:`.Operators.in_` method with a
        :class:`_sql.Select` object against a non-table-bound column would produce
        an ``AttributeError``, or more generally using a :class:`_sql.ScalarSelect`
        that has no datatype in a binary expression would produce invalid state.


    .. change::
        :tags: bug, mypy
        :tickets: sqlalchemy/sqlalchemy2-stubs/#14

        Fixed issue in mypy plugin where newly added support for
        :func:`_orm.as_declarative` needed to more fully add the
        ``DeclarativeMeta`` class to the mypy interpreter's state so that it does
        not result in a name not found error; additionally improves how global
        names are setup for the plugin including the ``Mapped`` name.


    .. change::
        :tags: bug, mysql, regression
        :tickets: 6163

        Fixed regression in the MySQL dialect where the reflection query used to
        detect if a table exists would fail on very old MySQL 5.0 and 5.1 versions.

    .. change::
        :tags: bug, sql
        :tickets: 6184

        Added a new flag to the :class:`_engine.Dialect` class called
        :attr:`_engine.Dialect.supports_statement_cache`. This flag now needs to be present
        directly on a dialect class in order for SQLAlchemy's
        :ref:`query cache <sql_caching>` to take effect for that dialect. The
        rationale is based on discovered issues such as :ticket:`6173` revealing
        that dialects which hardcode literal values from the compiled statement,
        often the numerical parameters used for LIMIT / OFFSET, will not be
        compatible with caching until these dialects are revised to use the
        parameters present in the statement only. For third party dialects where
        this flag is not applied, the SQL logging will show the message "dialect
        does not support caching", indicating the dialect should seek to apply this
        flag once they have verified that no per-statement literal values are being
        rendered within the compilation phase.

        .. seealso::

          :ref:`engine_thirdparty_caching`

    .. change::
        :tags: bug, postgresql
        :tickets: 6099

        Fixed typo in the fix for :ticket:`6099` released in 1.4.4 that completely
        prevented this change from working correctly, i.e. the error message did not match
        what was actually emitted by pg8000.

    .. change::
        :tags: bug, orm, regression
        :tickets: 6171

        Scaled back the warning message added in :ticket:`5171` to not warn for
        overlapping columns in an inheritance scenario where a particular
        relationship is local to a subclass and therefore does not represent an
        overlap.

    .. change::
        :tags: bug, regression, oracle
        :tickets: 6173

        Fixed critical regression where the Oracle compiler would not maintain the
        correct parameter values in the LIMIT/OFFSET for a select due to a caching
        issue.


    .. change::
        :tags: bug, postgresql
        :tickets: 6170

        Fixed issue where the PostgreSQL :class:`.PGInspector`, when generated
        against an :class:`_engine.Engine`, would fail for ``.get_enums()``,
        ``.get_view_names()``, ``.get_foreign_table_names()`` and
        ``.get_table_oid()`` when used against a "future" style engine and not the
        connection directly.

    .. change::
        :tags: bug, schema
        :tickets: 6146

        Introduce a new parameter :paramref:`_types.Enum.omit_aliases` in
        :class:`_types.Enum` type allow filtering aliases when using a pep435 Enum.
        Previous versions of SQLAlchemy kept aliases in all cases, creating
        database enum type with additional states, meaning that they were treated
        as different values in the db. For backward compatibility this flag
        defaults to ``False`` in the 1.4 series, but will be switched to ``True``
        in a future version. A deprecation warning is raise if this flag is not
        specified and the passed enum contains aliases.

    .. change::
        :tags: bug, mssql
        :tickets: 6163

        Fixed a regression in MSSQL 2012+ that prevented the order by clause
        to be rendered when ``offset=0`` is used in a subquery.

    .. change::
        :tags: bug, asyncio
        :tickets: 6166


        Fixed issue where the asyncio extension could not be loaded
        if running Python 3.6 with the backport library of
        ``contextvars`` installed.

.. changelog::
    :version: 1.4.4
    :released: March 30, 2021

    .. change::
        :tags: bug, misc

        Adjusted the usage of the ``importlib_metadata`` library for loading
        setuptools entrypoints in order to accommodate for some deprecation
        changes.


    .. change::
        :tags: bug, postgresql
        :tickets: 6099

        Modified the ``is_disconnect()`` handler for the pg8000 dialect, which now
        accommodates for a new ``InterfaceError`` emitted by pg8000 1.19.0. Pull
        request courtesy Hamdi Burak Usul.


    .. change::
        :tags: bug, orm
        :tickets: 6139

        Fixed critical issue in the new :meth:`_orm.PropComparator.and_` feature
        where loader strategies that emit secondary SELECT statements such as
        :func:`_orm.selectinload` and :func:`_orm.lazyload` would fail to
        accommodate for bound parameters in the user-defined criteria in terms of
        the current statement being executed, as opposed to the cached statement,
        causing stale bound values to be used.

        This also adds a warning for the case where an object that uses
        :func:`_orm.lazyload` in conjunction with :meth:`_orm.PropComparator.and_`
        is attempted to be serialized; the loader criteria cannot reliably
        be serialized and deserialized and eager loading should be used for this
        case.


    .. change::
        :tags: bug, engine
        :tickets: 6138

        Repair wrong arguments to exception handling method
        in CursorResult.

    .. change::
        :tags: bug, regression, orm
        :tickets: 6144

        Fixed missing method :meth:`_orm.Session.get` from the
        :class:`_orm.ScopedSession` interface.


    .. change::
        :tags: usecase, engine
        :tickets: 6155

        Modified the context manager used by :class:`_engine.Transaction` so that
        an "already detached" warning is not emitted by the ending of the context
        manager itself, if the transaction were already manually rolled back inside
        the block. This applies to regular transactions, savepoint transactions,
        and legacy "marker" transactions. A warning is still emitted if the
        ``.rollback()`` method is called explicitly more than once.

.. changelog::
    :version: 1.4.3
    :released: March 25, 2021

    .. change::
        :tags: bug, orm
        :tickets: 6069

        Fixed a bug where python 2.7.5 (default on CentOS 7) wasn't able to import
        sqlalchemy, because on this version of Python ``exec "statement"`` and
        ``exec("statement")`` do not behave the same way.  The compatibility
        ``exec_()`` function was used instead.

    .. change::
        :tags: sqlite, feature, asyncio
        :tickets: 5920

        Added support for the aiosqlite database driver for use with the
        SQLAlchemy asyncio extension.

        .. seealso::

          :ref:`aiosqlite`

    .. change::
        :tags: bug, regression, orm, declarative
        :tickets: 6128

        Fixed regression where the ``.metadata`` attribute on a per class level
        would not be honored, breaking the use case of per-class-hierarchy
        :class:`.schema.MetaData` for abstract declarative classes and mixins.


        .. seealso::

          :ref:`declarative_metadata`

    .. change::
        :tags: bug, mypy

        Added support for the Mypy extension to correctly interpret a declarative
        base class that's generated using the :func:`_orm.as_declarative` function
        as well as the :meth:`_orm.registry.as_declarative_base` method.

    .. change::
        :tags: bug, mypy
        :tickets: 6109

        Fixed bug in Mypy plugin where the Python type detection
        for the :class:`_types.Boolean` column type would produce
        an exception; additionally implemented support for :class:`_types.Enum`,
        including detection of a string-based enum vs. use of Python ``enum.Enum``.

    .. change::
        :tags: bug, reflection, postgresql
        :tickets: 6129

        Fixed reflection of identity columns in tables with mixed case names
        in PostgreSQL.

    .. change::
        :tags: bug, sqlite, regression
        :tickets: 5848

        Repaired the ``pysqlcipher`` dialect to connect correctly which had
        regressed in 1.4, and added test + CI support to maintain the driver
        in working condition.  The dialect now imports the ``sqlcipher3`` module
        for Python 3 by default before falling back to ``pysqlcipher3`` which
        is documented as now being unmaintained.

        .. seealso::

          :ref:`pysqlcipher`


    .. change::
        :tags: bug, orm
        :tickets: 6060

        Fixed bug where ORM queries using a correlated subquery in conjunction with
        :func:`_orm.column_property` would fail to correlate correctly to an
        enclosing subquery or to a CTE when :meth:`_sql.Select.correlate_except`
        were used in the property to control correlation, in cases where the
        subquery contained the same selectables as ones within the correlated
        subquery that were intended to not be correlated.

    .. change::
        :tags: bug, orm
        :tickets: 6131

        Fixed bug where combinations of the new "relationship with criteria"
        feature could fail in conjunction with features that make use of the new
        "lambda SQL" feature, including loader strategies such as selectinload and
        lazyload, for more complicated scenarios such as polymorphic loading.

    .. change::
        :tags: bug, orm
        :tickets: 6124

        Repaired support so that the :meth:`_sql.ClauseElement.params` method can
        work correctly with a :class:`_sql.Select` object that includes joins
        across ORM relationship structures, which is a new feature in 1.4.


    .. change::
        :tags: bug, engine, regression
        :tickets: 6119

        Restored the :class:`_engine.ResultProxy` name back to the
        ``sqlalchemy.engine`` namespace. This name refers to the
        :class:`_engine.LegacyCursorResult` object.

    .. change::
        :tags: bug, orm
        :tickets: 6115

        Fixed issue where a "removed in 2.0" warning were generated internally by
        the relationship loader mechanics.


.. changelog::
    :version: 1.4.2
    :released: March 19, 2021

    .. change::
        :tags: bug, orm, dataclasses
        :tickets: 6093

        Fixed issue in new ORM dataclasses functionality where dataclass fields on
        an abstract base or mixin that contained column or other mapping constructs
        would not be mapped if they also included a "default" key within the
        dataclasses.field() object.


    .. change::
        :tags: bug, regression, orm
        :tickets: 6088

        Fixed regression where the :attr:`_orm.Query.selectable` accessor, which is
        a synonym for :meth:`_orm.Query.__clause_element__`, got removed, it's now
        restored.

    .. change::
        :tags: bug, engine, regression

        Restored top level import for ``sqlalchemy.engine.reflection``. This
        ensures that the base :class:`_reflection.Inspector` class is properly
        registered so that :func:`_sa.inspect` works for third party dialects that
        don't otherwise import this package.


    .. change::
        :tags: bug, regression, orm
        :tickets: 6086

        Fixed regression where use of an unnamed SQL expression such as a SQL
        function would raise a column targeting error if the query itself were
        using joinedload for an entity and was also being wrapped in a subquery by
        the joinedload eager loading process.


    .. change::
        :tags: bug, orm, regression
        :tickets: 6092

        Fixed regression where the :meth:`_orm.Query.filter_by` method would fail
        to locate the correct source entity if the :meth:`_orm.Query.join` method
        had been used targeting an entity without any kind of ON clause.


    .. change::
        :tags: postgresql, usecase
        :tickets: 6982

        Rename the column name used by a reflection query that used
        a reserved word in some postgresql compatible databases.

    .. change::
        :tags: usecase, orm, dataclasses
        :tickets: 6100

        Added support for the :class:`_orm.declared_attr` object to work in the
        context of dataclass fields.

        .. seealso::

            :ref:`orm_declarative_dataclasses_mixin`

    .. change::
        :tags: bug, sql, regression
        :tickets: 6101

        Fixed issue where using a ``func`` that includes dotted packagenames would
        fail to be cacheable by the SQL caching system due to a Python list of
        names that needed to be a tuple.


    .. change::
        :tags: bug, regression, orm
        :tickets: 6095

        Fixed regression where the SQL compilation of a :class:`.Function` would
        not work correctly if the object had been "annotated", which is an internal
        memoization process used mostly by the ORM. In particular it could affect
        ORM lazy loads which make greater use of this feature in 1.4.

    .. change::
        :tags: bug, sql, regression
        :tickets: 6097

        Fixed regression in the :func:`_sql.case` construct, where the "dictionary"
        form of argument specification failed to work correctly if it were passed
        positionally, rather than as a "whens" keyword argument.

    .. change::
        :tags: bug, orm
        :tickets: 6090

        Fixed regression where the :class:`.ConcreteBase` would fail to map at all
        when a mapped column name overlapped with the discriminator column name,
        producing an assertion error. The use case here did not function correctly
        in 1.3 as the polymorphic union would produce a query that ignored the
        discriminator column entirely, while emitting duplicate column warnings. As
        1.4's architecture cannot easily reproduce this essentially broken behavior
        of 1.3 at the ``select()`` level right now, the use case now raises an
        informative error message instructing the user to use the
        ``.ConcreteBase._concrete_discriminator_name`` attribute to resolve the
        conflict. To assist with this configuration,
        ``.ConcreteBase._concrete_discriminator_name`` may be placed on the base
        class only where it will be automatically used by subclasses; previously
        this was not the case.


    .. change::
        :tags: bug, mypy
        :tickets: sqlalchemy/sqlalchemy2-stubs/2

        Fixed issue in MyPy extension which crashed on detecting the type of a
        :class:`.Column` if the type were given with a module prefix like
        ``sa.Integer()``.


.. changelog::
    :version: 1.4.1
    :released: March 17, 2021

    .. change::
        :tags: bug, orm, regression
        :tickets: 6066

        Fixed regression where producing a Core expression construct such as
        :func:`_sql.select` using ORM entities would eagerly configure the mappers,
        in an effort to maintain compatibility with the :class:`_orm.Query` object
        which necessarily does this to support many backref-related legacy cases.
        However, core :func:`_sql.select` constructs are also used in mapper
        configurations and such, and to that degree this eager configuration is
        more of an inconvenience, so eager configure has been disabled for the
        :func:`_sql.select` and other Core constructs in the absence of ORM loading
        types of functions such as :class:`_orm.Load`.

        The change maintains the behavior of :class:`_orm.Query` so that backwards
        compatibility is maintained. However, when using a :func:`_sql.select` in
        conjunction with ORM entities, a "backref" that isn't explicitly placed on
        one of the classes until mapper configure time won't be available unless
        :func:`_orm.configure_mappers` or the newer :func:`_orm.registry.configure`
        has been called elsewhere. Prefer using
        :paramref:`_orm.relationship.back_populates` for more explicit relationship
        configuration which does not have the eager configure requirement.


    .. change::
        :tags: bug, mssql, regression
        :tickets: 6058

        Fixed regression where a new setinputsizes() API that's available for
        pyodbc was enabled, which is apparently incompatible with pyodbc's
        fast_executemany() mode in the absence of more accurate typing information,
        which as of yet is not fully implemented or tested. The pyodbc dialect and
        connector has been modified so that setinputsizes() is not used at all
        unless the parameter ``use_setinputsizes`` is passed to the dialect, e.g.
        via :func:`_sa.create_engine`, at which point its behavior can be
        customized using the :meth:`.DialectEvents.do_setinputsizes` hook.

        .. seealso::

          :ref:`mssql_pyodbc_setinputsizes`

    .. change::
        :tags: bug, orm, regression
        :tickets: 6055

        Fixed a critical regression in the relationship lazy loader where the SQL
        criteria used to fetch a related many-to-one object could go stale in
        relation to other memoized structures within the loader if the mapper had
        configuration changes, such as can occur when mappers are late configured
        or configured on demand, producing a comparison to None and returning no
        object. Huge thanks to Alan Hamlett for their help tracking this down late
        into the night.



    .. change::
        :tags: bug, regression
        :tickets: 6068

        Added back ``items`` and ``values`` to ``ColumnCollection`` class.
        The regression was introduced while adding support for duplicate
        columns in from clauses and selectable in ticket #4753.


    .. change::
        :tags: bug, engine, regression
        :tickets: 6074

        The Python ``namedtuple()`` has the behavior such that the names ``count``
        and ``index`` will be served as tuple values if the named tuple includes
        those names; if they are absent, then their behavior as methods of
        ``collections.abc.Sequence`` is maintained. Therefore the
        :class:`_result.Row` and :class:`_result.LegacyRow` classes have been fixed
        so that they work in this same way, maintaining the expected behavior for
        database rows that have columns named "index" or "count".

    .. change::
        :tags: bug, orm, regression
        :tickets: 6076

        Fixed regression where the :meth:`_orm.Query.exists` method would fail to
        create an expression if the entity list of the :class:`_orm.Query` were
        an arbitrary SQL column expression.


    .. change::
        :tags: bug, orm, regression
        :tickets: 6052

        Fixed regression where calling upon :meth:`_orm.Query.count` in conjunction
        with a loader option such as :func:`_orm.joinedload` would fail to ignore
        the loader option. This is a behavior that has always been very specific to
        the :meth:`_orm.Query.count` method; an error is normally raised if a given
        :class:`_orm.Query` has options that don't apply to what it is returning.

    .. change::
        :tags: bug, orm, declarative, regression
        :tickets: 6054

        Fixed bug where user-mapped classes that contained an attribute named
        "registry" would cause conflicts with the new registry-based mapping system
        when using :class:`.DeclarativeMeta`. While the attribute remains
        something that can be set explicitly on a declarative base to be
        consumed by the metaclass, once located it is placed under a private
        class variable so it does not conflict with future subclasses that use
        the same name for other purposes.



    .. change::
        :tags: bug, orm, regression
        :tickets: 6067

        Fixed regression in :meth:`_orm.Session.identity_key`, including that the
        method and related methods were not covered by any unit test as well as
        that the method contained a typo preventing it from functioning correctly.


.. changelog::
    :version: 1.4.0
    :released: March 15, 2021

    .. change::
        :tags: bug, mssql
        :tickets: 5919

        Fix a reflection error for MSSQL 2005 introduced by the reflection of
        filtered indexes.

    .. change::
        :tags: feature, mypy
        :tickets: 4609

        Rudimentary and experimental support for Mypy has been added in the form of
        a new plugin, which itself depends on new typing stubs for SQLAlchemy. The
        plugin allows declarative mappings in their standard form to both be
        compatible with Mypy as well as to provide typing support for mapped
        classes and instances.

        .. seealso::

            :ref:`mypy_toplevel`

    .. change::
        :tags: bug, sql
        :tickets: 6016

        Fixed bug where the "percent escaping" feature that occurs with dialects
        that use the "format" or "pyformat" bound parameter styles was not enabled
        for the :meth:`_sql.Operators.op` and :class:`_sql.custom_op` constructs,
        for custom operators that use percent signs. The percent sign will now be
        automatically doubled based on the paramstyle as necessary.



    .. change::
        :tags: bug, regression, sql
        :tickets: 5979

        Fixed regression where the "unsupported compilation error" for unknown
        datatypes would fail to raise correctly.

    .. change::
        :tags: ext, usecase
        :tickets: 5942

        Add new parameter
        :paramref:`_automap.AutomapBase.prepare.reflection_options`
        to allow passing of :meth:`_schema.MetaData.reflect` options like ``only``
        or dialect-specific reflection options like ``oracle_resolve_synonyms``.

    .. change::
        :tags: change, sql

        Altered the compilation for the :class:`.CTE` construct so that a string is
        returned representing the inner SELECT statement if the :class:`.CTE` is
        stringified directly, outside of the context of an enclosing SELECT; This
        is the same behavior of :meth:`_sql.FromClause.alias` and
        :meth:`_sql.Select.subquery`. Previously, a blank string would be
        returned as the CTE is normally placed above a SELECT after that SELECT has
        been generated, which is generally misleading when debugging.


    .. change::
        :tags: bug, orm
        :tickets: 5981

        Fixed regression where the :paramref:`_orm.relationship.query_class`
        parameter stopped being functional for "dynamic" relationships.  The
        ``AppenderQuery`` remains dependent on the legacy :class:`_orm.Query`
        class; users are encouraged to migrate from the use of "dynamic"
        relationships to using :func:`_orm.with_parent` instead.


    .. change::
        :tags: bug, orm, regression
        :tickets: 6003

        Fixed regression where :meth:`_orm.Query.join` would produce no effect if
        the query itself as well as the join target were against a
        :class:`_schema.Table` object, rather than a mapped class. This was part of
        a more systemic issue where the legacy ORM query compiler would not be
        correctly used from a :class:`_orm.Query` if the statement produced had not
        ORM entities present within it.


    .. change::
        :tags: bug, regression, sql
        :tickets: 6008

        Fixed regression where usage of the standalone :func:`_sql.distinct()` used
        in the form of being directly SELECTed would fail to be locatable in the
        result set by column identity, which is how the ORM locates columns. While
        standalone :func:`_sql.distinct()` is not oriented towards being directly
        SELECTed (use :meth:`_sql.select.distinct` for a regular
        ``SELECT DISTINCT..``) , it was usable to a limited extent in this way
        previously (but wouldn't work in subqueries, for example). The column
        targeting for unary expressions such as "DISTINCT <col>" has been improved
        so that this case works again, and an additional improvement has been made
        so that usage of this form in a subquery at least generates valid SQL which
        was not the case previously.

        The change additionally enhances the ability to target elements in
        ``row._mapping`` based on SQL expression objects in ORM-enabled
        SELECT statements, including whether the statement was invoked by
        ``connection.execute()`` or ``session.execute()``.

    .. change::
        :tags: bug, orm, asyncio
        :tickets: 5998

        The API for :meth:`_asyncio.AsyncSession.delete` is now an awaitable;
        this method cascades along relationships which must be loaded in a
        similar manner as the :meth:`_asyncio.AsyncSession.merge` method.


    .. change::
        :tags: usecase, postgresql, mysql, asyncio
        :tickets: 5967

        Added an ``asyncio.Lock()`` within SQLAlchemy's emulated DBAPI cursor,
        local to the connection, for the asyncpg and aiomysql dialects for the
        scope of the ``cursor.execute()`` and ``cursor.executemany()`` methods. The
        rationale is to prevent failures and corruption for the case where the
        connection is used in multiple awaitables at once.

        While this use case can also occur with threaded code and non-asyncio
        dialects, we anticipate this kind of use will be more common under asyncio,
        as the asyncio API is encouraging of such use. It's definitely better to
        use a distinct connection per concurrent awaitable however as concurrency
        will not be achieved otherwise.

        For the asyncpg dialect, this is so that the space between
        the call to ``prepare()`` and ``fetch()`` is prevented from allowing
        concurrent executions on the connection from causing interface error
        exceptions, as well as preventing race conditions when starting a new
        transaction. Other PostgreSQL DBAPIs are threadsafe at the connection level
        so this intends to provide a similar behavior, outside the realm of server
        side cursors.

        For the aiomysql dialect, the mutex will provide safety such that
        the statement execution and the result set fetch, which are two distinct
        steps at the connection level, won't get corrupted by concurrent
        executions on the same connection.


    .. change::
        :tags: bug, engine
        :tickets: 6002

        Improved engine logging to note ROLLBACK and COMMIT which is logged while
        the DBAPI driver is in AUTOCOMMIT mode. These ROLLBACK/COMMIT are library
        level and do not have any effect when AUTOCOMMIT is in effect, however it's
        still worthwhile to log as these indicate where SQLAlchemy sees the
        "transaction" demarcation.

    .. change::
        :tags: bug, regression, engine
        :tickets: 6004

        Fixed a regression where the "reset agent" of the connection pool wasn't
        really being utilized by the :class:`_engine.Connection` when it were
        closed, and also leading to a double-rollback scenario that was somewhat
        wasteful.   The newer architecture of the engine has been updated so that
        the connection pool "reset-on-return" logic will be skipped when the
        :class:`_engine.Connection` explicitly closes out the transaction before
        returning the pool to the connection.

    .. change::
        :tags: bug, schema
        :tickets: 5953

        Deprecated all schema-level ``.copy()`` methods and renamed to
        ``_copy()``.  These are not standard Python "copy()" methods as they
        typically rely upon being instantiated within particular contexts
        which are passed to the method as optional keyword arguments.   The
        :meth:`_schema.Table.tometadata` method is the public API that provides
        copying for :class:`_schema.Table` objects.

    .. change::
        :tags: bug, ext
        :tickets: 6020

        The ``sqlalchemy.ext.mutable`` extension now tracks the "parents"
        collection using the :class:`.InstanceState` associated with objects,
        rather than the object itself. The latter approach required that the object
        be hashable so that it can be inside of a ``WeakKeyDictionary``, which goes
        against the behavioral contract of the ORM overall which is that ORM mapped
        objects do not need to provide any particular kind of ``__hash__()`` method
        and that unhashable objects are supported.

    .. change::
        :tags: bug, orm
        :tickets: 5984

        The unit of work process now turns off all "lazy='raise'" behavior
        altogether when a flush is proceeding.  While there are areas where the UOW
        is sometimes loading things that aren't ultimately needed, the lazy="raise"
        strategy is not helpful here as the user often does not have much control
        or visibility into the flush process.


.. changelog::
    :version: 1.4.0b3
    :released: March 15, 2021
    :released: February 15, 2021

    .. change::
        :tags: bug, orm
        :tickets: 5933

        Fixed issue in new 1.4/2.0 style ORM queries where a statement-level label
        style would not be preserved in the keys used by result rows; this has been
        applied to all combinations of Core/ORM columns / session vs. connection
        etc. so that the linkage from statement to result row is the same in all
        cases.   As part of this change, the labeling of column expressions
        in rows has been improved to retain the original name of the ORM
        attribute even if used in a subquery.




    .. change::
        :tags: bug, sql
        :tickets: 5924

        Fixed bug where the "cartesian product" assertion was not correctly
        accommodating for joins between tables that relied upon the use of LATERAL
        to connect from a subquery to another subquery in the enclosing context.

    .. change::
        :tags: bug, sql
        :tickets: 5934

        Fixed 1.4 regression where the :meth:`_functions.Function.in_` method was
        not covered by tests and failed to function properly in all cases.

    .. change::
        :tags: bug, engine, postgresql
        :tickets: 5941

        Continued with the improvement made as part of :ticket:`5653` to further
        support bound parameter names, including those generated against column
        names, for names that include colons, parenthesis, and question marks, as
        well as improved test support, so that bound parameter names even if they
        are auto-derived from column names should have no problem including for
        parenthesis in psycopg2's "pyformat" style.

        As part of this change, the format used by the asyncpg DBAPI adapter (which
        is local to SQLAlchemy's asyncpg dialect) has been changed from using
        "qmark" paramstyle to "format", as there is a standard and internally
        supported SQL string escaping style for names that use percent signs with
        "format" style (i.e. to double percent signs), as opposed to names that use
        question marks with "qmark" style (where an escaping system is not defined
        by pep-249 or Python).

        .. seealso::

          :ref:`change_5941`

    .. change::
        :tags: sql, usecase, postgresql, sqlite
        :tickets: 5939

        Enhance ``set_`` keyword of :class:`.OnConflictDoUpdate` to accept a
        :class:`.ColumnCollection`, such as the ``.c.`` collection from a
        :class:`Selectable`, or the ``.excluded`` contextual object.

    .. change::
        :tags: feature, orm

        The ORM used in :term:`2.0 style` can now return ORM objects from the rows
        returned by an UPDATE..RETURNING or INSERT..RETURNING statement, by
        supplying the construct to :meth:`_sql.Select.from_statement` in an ORM
        context.

        .. seealso::

          :ref:`orm_dml_returning_objects`



    .. change::
        :tags: bug, sql
        :tickets: 5935

        Fixed regression where use of an arbitrary iterable with the
        :func:`_sql.select` function was not working, outside of plain lists. The
        forwards/backwards compatibility logic here now checks for a wider range of
        incoming "iterable" types including that a ``.c`` collection from a
        selectable can be passed directly. Pull request compliments of Oliver Rice.

.. changelog::
    :version: 1.4.0b2
    :released: March 15, 2021
    :released: February 3, 2021

    .. change::
        :tags: usecase, sql
        :tickets: 5695

        Multiple calls to "returning", e.g. :meth:`_sql.Insert.returning`,
        may now be chained to add new columns to the RETURNING clause.


    .. change::
      :tags: bug, asyncio
      :tickets: 5615

      Adjusted the greenlet integration, which provides support for Python asyncio
      in SQLAlchemy, to accommodate for the handling of Python ``contextvars``
      (introduced in Python 3.7) for ``greenlet`` versions greater than 0.4.17.
      Greenlet version 0.4.17 added automatic handling of contextvars in a
      backwards-incompatible way; we've coordinated with the greenlet authors to
      add a preferred API for this in versions subsequent to 0.4.17 which is now
      supported by SQLAlchemy's greenlet integration.  For greenlet versions prior
      to 0.4.17 no behavioral change is needed, version 0.4.17 itself is blocked
      from the dependencies.

    .. change::
        :tags: bug, engine, sqlite
        :tickets: 5845

        Fixed bug in the 2.0 "future" version of :class:`.Engine` where emitting
        SQL during the :meth:`.EngineEvents.begin` event hook would cause a
        re-entrant (recursive) condition due to autobegin, affecting among other
        things the recipe documented for SQLite to allow for savepoints and
        serializable isolation support.


    .. change::
        :tags: bug, orm, regression
        :tickets: 5845

        Fixed issue in new :class:`_orm.Session` similar to that of the
        :class:`_engine.Connection` where the new "autobegin" logic could be
        tripped into a re-entrant (recursive) state if SQL were executed within the
        :meth:`.SessionEvents.after_transaction_create` event hook.

    .. change::
        :tags: sql
        :tickets: 4757

        Replace :meth:`_orm.Query.with_labels` and
        :meth:`_sql.GenerativeSelect.apply_labels` with explicit getters and
        setters :meth:`_sql.GenerativeSelect.get_label_style` and
        :meth:`_sql.GenerativeSelect.set_label_style` to accommodate the three
        supported label styles: :data:`_sql.LABEL_STYLE_DISAMBIGUATE_ONLY`,
        :data:`_sql.LABEL_STYLE_TABLENAME_PLUS_COL`, and
        :data:`_sql.LABEL_STYLE_NONE`.

        In addition, for Core and "future style" ORM queries,
        ``LABEL_STYLE_DISAMBIGUATE_ONLY`` is now the default label style. This
        style differs from the existing "no labels" style in that labeling is
        applied in the case of column name conflicts; with ``LABEL_STYLE_NONE``, a
        duplicate column name is not accessible via name in any case.

        For cases where labeling is significant, namely that the ``.c`` collection
        of a subquery is able to refer to all columns unambiguously, the behavior
        of ``LABEL_STYLE_DISAMBIGUATE_ONLY`` is now sufficient for all
        SQLAlchemy features across Core and ORM which involve this behavior.
        Result set rows since SQLAlchemy 1.0 are usually aligned with column
        constructs positionally.

        For legacy ORM queries using :class:`_query.Query`, the table-plus-column
        names labeling style applied by ``LABEL_STYLE_TABLENAME_PLUS_COL``
        continues to be used so that existing test suites and logging facilities
        see no change in behavior by default.

    .. change::
        :tags: bug, orm, unitofwork
        :tickets: 5735

        Improved the unit of work topological sorting system such that the
        toplogical sort is now deterministic based on the sorting of the input set,
        which itself is now sorted at the level of mappers, so that the same inputs
        of affected mappers should produce the same output every time, among
        mappers / tables that don't have any dependency on each other. This further
        reduces the chance of deadlocks as can be observed in a flush that UPDATEs
        among multiple, unrelated tables such that row locks are generated.


    .. change::
        :tags: changed, orm
        :tickets: 5897

        Mapper "configuration", which occurs within the
        :func:`_orm.configure_mappers` function, is now organized to be on a
        per-registry basis. This allows for example the mappers within a certain
        declarative base to be configured, but not those of another base that is
        also present in memory. The goal is to provide a means of reducing
        application startup time by only running the "configure" process for sets
        of mappers that are needed. This also adds the
        :meth:`_orm.registry.configure` method that will run configure for the
        mappers local in a particular registry only.

    .. change::
        :tags: bug, orm
        :tickets: 5702

        Fixed regression where the :paramref:`.Bundle.single_entity` flag would
        take effect for a :class:`.Bundle` even though it were not set.
        Additionally, this flag is legacy as it only makes sense for the
        :class:`_orm.Query` object and not 2.0 style execution.  a deprecation
        warning is emitted when used with new-style execution.

    .. change::
        :tags: bug, sql
        :tickets: 5858

        Fixed issue in new :meth:`_sql.Select.join` method where chaining from the
        current JOIN wasn't looking at the right state, causing an expression like
        "FROM a JOIN b <onclause>, b JOIN c <onclause>" rather than
        "FROM a JOIN b <onclause> JOIN c <onclause>".

    .. change::
        :tags: usecase, sql

        Added :meth:`_sql.Select.outerjoin_from` method to complement
        :meth:`_sql.Select.join_from`.

    .. change::
        :tags: usecase, sql
        :tickets: 5888

        Adjusted the "literal_binds" feature of :class:`_sql.Compiler` to render
        NULL for a bound parameter that has ``None`` as the value, either
        explicitly passed or omitted. The previous error message "bind parameter
        without a renderable value" is removed, and a missing or ``None`` value
        will now render NULL in all cases. Previously, rendering of NULL was
        starting to happen for DML statements due to internal refactorings, but was
        not explicitly part of test coverage, which it now is.

        While no error is raised, when the context is within that of a column
        comparison, and the operator is not "IS"/"IS NOT", a warning is emitted
        that this is not generally useful from a SQL perspective.


    .. change::
        :tags: bug, orm
        :tickets: 5750

        Fixed regression where creating an :class:`_orm.aliased` construct against
        a plain selectable and including a name would raise an assertionerror.


    .. change::
        :tags: bug, mssql, mysql, datatypes
        :tickets: 5788
        :versions: 1.4.0b2

        Decimal accuracy and behavior has been improved when extracting floating
        point and/or decimal values from JSON strings using the
        :meth:`_sql.sqltypes.JSON.Comparator.as_float` method, when the numeric
        value inside of the JSON string has many significant digits; previously,
        MySQL backends would truncate values with many significant digits and SQL
        Server backends would raise an exception due to a DECIMAL cast with
        insufficient significant digits.   Both backends now use a FLOAT-compatible
        approach that does not hardcode significant digits for floating point
        values. For precision numerics, a new method
        :meth:`_sql.sqltypes.JSON.Comparator.as_numeric` has been added which
        accepts arguments for precision and scale, and will return values as Python
        ``Decimal`` objects with no floating point conversion assuming the DBAPI
        supports it (all but pysqlite).

    .. change::
        :tags: feature, orm, declarative
        :tickets: 5745

        Added an alternate resolution scheme to Declarative that will extract the
        SQLAlchemy column or mapped property from the "metadata" dictionary of a
        dataclasses.Field object.  This allows full declarative mappings to be
        combined with dataclass fields.

        .. seealso::

            :ref:`orm_declarative_dataclasses_declarative_table`

    .. change::
        :tags: bug, sql
        :tickets: 5754

        Deprecation warnings are emitted under "SQLALCHEMY_WARN_20" mode when
        passing a plain string to :meth:`_orm.Session.execute`.


    .. change::
        :tags: bug, sql, orm
        :tickets: 5760, 5763, 5765, 5768, 5770

        A wide variety of fixes to the "lambda SQL" feature introduced at
        :ref:`engine_lambda_caching` have been implemented based on user feedback,
        with an emphasis on its use within the :func:`_orm.with_loader_criteria`
        feature where it is most prominently used [ticket:5760]:

        * fixed issue where boolean True/False values referred towards in the
          closure variables of the lambda would cause failures [ticket:5763]

        * Repaired a non-working detection for Python functions embedded in the
          lambda that produce bound values; this case is likely not supportable
          so raises an informative error, where the function should be invoked
          outside the lambda itself.  New documentation has been added to
          further detail this behavior. [ticket:5770]

        * The lambda system by default now rejects the use of non-SQL elements
          within the closure variables of the lambda entirely, where the error
          suggests the two options of either explicitly ignoring closure variables
          that are not SQL parameters, or specifying a specific set of values to be
          considered as part of the cache key based on hash value.   This critically
          prevents the lambda system from assuming that arbitrary objects within
          the lambda's closure are appropriate for caching while also refusing to
          ignore them by default, preventing the case where their state might
          not be constant and have an impact on the SQL construct produced.
          The error message is comprehensive and new documentation has been
          added to further detail this behavior. [ticket:5765]

        * Fixed support for the edge case where an ``in_()`` expression
          against a list of SQL elements, such as :func:`_sql.literal` objects,
          would fail to be accommodated correctly. [ticket:5768]


    .. change::
        :tags: bug, orm
        :tickets: 5760, 5766, 5762, 5761, 5764

        Related to the fixes for the lambda criteria system within Core, within the
        ORM implemented a variety of fixes for the
        :func:`_orm.with_loader_criteria` feature as well as the
        :meth:`_orm.SessionEvents.do_orm_execute` event handler that is often
        used in conjunction [ticket:5760]:


        * fixed issue where :func:`_orm.with_loader_criteria` function would fail
          if the given entity or base included non-mapped mixins in its descending
          class hierarchy [ticket:5766]

        * The :func:`_orm.with_loader_criteria` feature is now unconditionally
          disabled for the case of ORM "refresh" operations, including loads
          of deferred or expired column attributes as well as for explicit
          operations like :meth:`_orm.Session.refresh`.  These loads are necessarily
          based on primary key identity where addiional WHERE criteria is
          never appropriate.  [ticket:5762]

        * Added new attribute :attr:`_orm.ORMExecuteState.is_column_load` to indicate
          that a :meth:`_orm.SessionEvents.do_orm_execute` handler that a particular
          operation is a primary-key-directed column attribute load, where additional
          criteria should not be added.  The :func:`_orm.with_loader_criteria`
          function as above ignores these in any case now.  [ticket:5761]

        * Fixed issue where the :attr:`_orm.ORMExecuteState.is_relationship_load`
          attribute would not be set correctly for many lazy loads as well as all
          selectinloads.  The flag is essential in order to test if options should
          be added to statements or if they would already have been propagated via
          relationship loads.  [ticket:5764]


    .. change::
        :tags: usecase, orm

        Added :attr:`_orm.ORMExecuteState.bind_mapper` and
        :attr:`_orm.ORMExecuteState.all_mappers` accessors to
        :class:`_orm.ORMExecuteState` event object, so that handlers can respond to
        the target mapper and/or mapped class or classes involved in an ORM
        statement execution.

    .. change::
        :tags: bug, engine, postgresql, oracle

        Adjusted the "setinputsizes" logic relied upon by the cx_Oracle, asyncpg
        and pg8000 dialects to support a :class:`.TypeDecorator` that includes
        an override the :meth:`.TypeDecorator.get_dbapi_type()` method.


    .. change::
        :tags: postgresql, performance

        Enhanced the performance of the asyncpg dialect by caching the asyncpg
        PreparedStatement objects on a per-connection basis. For a test case that
        makes use of the same statement on a set of pooled connections this appears
        to grant a 10-20% speed improvement.  The cache size is adjustable and may
        also be disabled.

        .. seealso::

            :ref:`asyncpg_prepared_statement_cache`

    .. change::
        :tags: feature, mysql
        :tickets: 5747

        Added support for the aiomysql driver when using the asyncio SQLAlchemy
        extension.

        .. seealso::

          :ref:`aiomysql`

    .. change::
        :tags: bug, reflection
        :tickets: 5684

        Fixed bug where the now-deprecated ``autoload`` parameter was being called
        internally within the reflection routines when a related table were
        reflected.


    .. change::
        :tags: platform, performance
        :tickets: 5681

        Adjusted some elements related to internal class production at import time
        which added significant latency to the time spent to import the library vs.
        that of 1.3.   The time is now about 20-30% slower than 1.3 instead of
        200%.


    .. change::
        :tags: changed, schema
        :tickets: 5775

        Altered the behavior of the :class:`_schema.Identity` construct such that
        when applied to a :class:`_schema.Column`, it will automatically imply that
        the value of :paramref:`_sql.Column.nullable` should default to ``False``,
        in a similar manner as when the :paramref:`_sql.Column.primary_key`
        parameter is set to ``True``.   This matches the default behavior of all
        supporting databases where ``IDENTITY`` implies ``NOT NULL``.  The
        PostgreSQL backend is the only one that supports adding ``NULL`` to an
        ``IDENTITY`` column, which is here supported by passing a ``True`` value
        for the :paramref:`_sql.Column.nullable` parameter at the same time.


    .. change::
        :tags: bug, postgresql
        :tickets: 5698

        Fixed a small regression where the query for "show
        standard_conforming_strings" upon initialization would be emitted even if
        the server version info were detected as less than version 8.2, previously
        it would only occur for server version 8.2 or greater. The query fails on
        Amazon Redshift which reports a PG server version older than this value.


    .. change::
        :tags: bug, sql, postgresql, mysql, sqlite
        :tickets: 5169

        An informative error message is now raised for a selected set of DML
        methods (currently all part of :class:`_dml.Insert` constructs) if they are
        called a second time, which would implicitly cancel out the previous
        setting.  The methods altered include:
        :class:`_sqlite.Insert.on_conflict_do_update`,
        :class:`_sqlite.Insert.on_conflict_do_nothing` (SQLite),
        :class:`_postgresql.Insert.on_conflict_do_update`,
        :class:`_postgresql.Insert.on_conflict_do_nothing` (PostgreSQL),
        :class:`_mysql.Insert.on_duplicate_key_update` (MySQL)

    .. change::
        :tags: pool, tests, usecase
        :tickets: 5582

        Improve documentation and add test for sub-second pool timeouts.
        Pull request courtesy Jordan Pittier.

    .. change::
        :tags: bug, general

        Fixed a SQLite source file that had non-ascii characters inside of its
        docstring without a source encoding, introduced within the "INSERT..ON
        CONFLICT" feature, which would cause failures under Python 2.

    .. change::
        :tags: sqlite, usecase
        :tickets: 4010

        Implemented INSERT... ON CONFLICT clause for SQLite. Pull request courtesy
        Ramon Williams.

        .. seealso::

            :ref:`sqlite_on_conflict_insert`

    .. change::
        :tags: bug, asyncio
        :tickets: 5811

        Implemented "connection-binding" for :class:`.AsyncSession`, the ability to
        pass an :class:`.AsyncConnection` to create an :class:`.AsyncSession`.
        Previously, this use case was not implemented and would use the associated
        engine when the connection were passed.  This fixes the issue where the
        "join a session to an external transaction" use case would not work
        correctly for the :class:`.AsyncSession`.  Additionally, added methods
        :meth:`.AsyncConnection.in_transaction`,
        :meth:`.AsyncConnection.in_nested_transaction`,
        :meth:`.AsyncConnection.get_transaction`,
        :meth:`.AsyncConnection.get_nested_transaction` and
        :attr:`.AsyncConnection.info` attribute.

    .. change::
        :tags: usecase, asyncio

        The :class:`.AsyncEngine`, :class:`.AsyncConnection` and
        :class:`.AsyncTransaction` objects may be compared using Python ``==`` or
        ``!=``, which will compare the two given objects based on the "sync" object
        they are proxying towards. This is useful as there are cases particularly
        for :class:`.AsyncTransaction` where multiple instances of
        :class:`.AsyncTransaction` can be proxying towards the same sync
        :class:`_engine.Transaction`, and are actually equivalent.   The
        :meth:`.AsyncConnection.get_transaction` method will currently return a new
        proxying :class:`.AsyncTransaction` each time as the
        :class:`.AsyncTransaction` is not otherwise statefully associated with its
        originating :class:`.AsyncConnection`.

    .. change::
        :tags: bug, oracle
        :tickets: 5884

        Oracle two-phase transactions at a rudimentary level are now no longer
        deprecated. After receiving support from cx_Oracle devs we can provide for
        basic xid + begin/prepare support with some limitations, which will work
        more fully in an upcoming release of cx_Oracle. Two phase "recovery" is not
        currently supported.

    .. change::
        :tags: asyncio

        The SQLAlchemy async mode now detects and raises an informative
        error when an non asyncio compatible :term:`DBAPI` is used.
        Using a standard ``DBAPI`` with async SQLAlchemy will cause
        it to block like any sync call, interrupting the executing asyncio
        loop.

    .. change::
        :tags: usecase, orm, asyncio
        :tickets: 5796, 5797, 5802

        Added :meth:`_asyncio.AsyncSession.scalar`,
        :meth:`_asyncio.AsyncSession.get` as well as support for
        :meth:`_orm.sessionmaker.begin` to work as an async context manager with
        :class:`_asyncio.AsyncSession`.  Also added
        :meth:`_asyncio.AsyncSession.in_transaction` accessor.

    .. change::
        :tags: bug, sql
        :tickets: 5785

        Fixed issue in new :class:`_sql.Values` construct where passing tuples of
        objects would fall back to per-value type detection rather than making use
        of the :class:`_schema.Column` objects passed directly to
        :class:`_sql.Values` that tells SQLAlchemy what the expected type is. This
        would lead to issues for objects such as enumerations and numpy strings
        that are not actually necessary since the expected type is given.

    .. change::
        :tags: bug, engine

        Added the "future" keyword to the list of words that are known by the
        :func:`_sa.engine_from_config` function, so that the values "true" and
        "false" may be configured as "boolean" values when using a key such
        as ``sqlalchemy.future = true`` or ``sqlalchemy.future = false``.


    .. change::
        :tags: usecase, schema
        :tickets: 5712

        The :meth:`_events.DDLEvents.column_reflect` event may now be applied to a
        :class:`_schema.MetaData` object where it will take effect for the
        :class:`_schema.Table` objects local to that collection.

        .. seealso::

            :meth:`_events.DDLEvents.column_reflect`

            :ref:`mapper_automated_reflection_schemes` - in the ORM mapping documentation

            :ref:`automap_intercepting_columns` - in the :ref:`automap_toplevel` documentation




    .. change::
        :tags: feature, engine

        Dialect-specific constructs such as
        :meth:`_postgresql.Insert.on_conflict_do_update` can now stringify in-place
        without the need to specify an explicit dialect object.  The constructs,
        when called upon for ``str()``, ``print()``, etc. now have internal
        direction to call upon their appropriate dialect rather than the
        "default"dialect which doesn't know how to stringify these.   The approach
        is also adapted to generic schema-level create/drop such as
        :class:`_schema.AddConstraint`, which will adapt its stringify dialect to
        one indicated by the element within it, such as the
        :class:`_postgresql.ExcludeConstraint` object.


    .. change::
        :tags: feature, engine
        :tickets: 5911

        Added new execution option
        :paramref:`_engine.Connection.execution_options.logging_token`. This option
        will add an additional per-message token to log messages generated by the
        :class:`_engine.Connection` as it executes statements. This token is not
        part of the logger name itself (that part can be affected using the
        existing :paramref:`_sa.create_engine.logging_name` parameter), so is
        appropriate for ad-hoc connection use without the side effect of creating
        many new loggers. The option can be set at the level of
        :class:`_engine.Connection` or :class:`_engine.Engine`.

        .. seealso::

          :ref:`dbengine_logging_tokens`

    .. change::
        :tags: bug, pool
        :tickets: 5708

        Fixed regression where a connection pool event specified with a keyword,
        most notably ``insert=True``, would be lost when the event were set up.
        This would prevent startup events that need to fire before dialect-level
        events from working correctly.


    .. change::
        :tags: usecase, pool
        :tickets: 5708, 5497

        The internal mechanics of the engine connection routine has been altered
        such that it's now guaranteed that a user-defined event handler for the
        :meth:`_pool.PoolEvents.connect` handler, when established using
        ``insert=True``, will allow an event handler to run that is definitely
        invoked **before** any dialect-specific initialization starts up, most
        notably when it does things like detect default schema name.
        Previously, this would occur in most cases but not unconditionally.
        A new example is added to the schema documentation illustrating how to
        establish the "default schema name" within an on-connect event.

    .. change::
        :tags: usecase, postgresql

        Added a read/write ``.autocommit`` attribute to the DBAPI-adaptation layer
        for the asyncpg dialect.   This so that when working with DBAPI-specific
        schemes that need to use "autocommit" directly with the DBAPI connection,
        the same ``.autocommit`` attribute which works with both psycopg2 as well
        as pg8000 is available.

    .. change::
        :tags: bug, oracle
        :tickets: 5716

        The Oracle dialect now uses
        ``select sys_context( 'userenv', 'current_schema' ) from dual`` to get
        the default schema name, rather than ``SELECT USER FROM DUAL``, to
        accommodate for changes to the session-local schema name under Oracle.

    .. change::
        :tags: schema, feature
        :tickets: 5659

        Added :meth:`_types.TypeEngine.as_generic` to map dialect-specific types,
        such as :class:`sqlalchemy.dialects.mysql.INTEGER`, with the "best match"
        generic SQLAlchemy type, in this case :class:`_types.Integer`.  Pull
        request courtesy Andrew Hannigan.

        .. seealso::

          :ref:`metadata_reflection_dbagnostic_types` - example usage

    .. change::
        :tags: bug, sql
        :tickets: 5717

        Fixed issue where a :class:`.RemovedIn20Warning` would erroneously emit
        when the ``.bind`` attribute were accessed internally on objects,
        particularly when stringifying a SQL construct.

    .. change::
        :tags: bug, orm
        :tickets: 5781

        Fixed 1.4 regression where the use of :meth:`_orm.Query.having` in
        conjunction with queries with internally adapted SQL elements (common in
        inheritance scenarios) would fail due to an incorrect function call. Pull
        request courtesy esoh.


    .. change::
        :tags: bug, pool, pypy
        :tickets: 5842

        Fixed issue where connection pool would not return connections to the pool
        or otherwise be finalized upon garbage collection under pypy if the checked
        out connection fell out of scope without being closed.   This is a long
        standing issue due to pypy's difference in GC behavior that does not call
        weakref finalizers if they are relative to another object that is also
        being garbage collected.  A strong reference to the related record is now
        maintained so that the weakref has a strong-referenced "base" to trigger
        off of.

    .. change::
        :tags: bug, sqlite
        :tickets: 5699

        Use python ``re.search()`` instead of ``re.match()`` as the operation
        used by the :meth:`Column.regexp_match` method when using sqlite.
        This matches the behavior of regular expressions on other databases
        as well as that of well-known SQLite plugins.

    .. change::
        :tags: changed, postgresql

        Fixed issue where the psycopg2 dialect would silently pass the
        ``use_native_unicode=False`` flag without actually having any effect under
        Python 3, as the psycopg2 DBAPI uses Unicode unconditionally under Python
        3.  This usage now raises an :class:`_exc.ArgumentError` when used under
        Python 3. Added test support for Python 2.

    .. change::
        :tags: bug, postgresql
        :tickets: 5722
        :versions: 1.4.0b2

        Established support for :class:`_schema.Column` objects as well as ORM
        instrumented attributes as keys in the ``set_`` dictionary passed to the
        :meth:`_postgresql.Insert.on_conflict_do_update` and
        :meth:`_sqlite.Insert.on_conflict_do_update` methods, which match to the
        :class:`_schema.Column` objects in the ``.c`` collection of the target
        :class:`_schema.Table`. Previously,  only string column names were
        expected; a column expression would be assumed to be an out-of-table
        expression that would render fully along with a warning.

    .. change::
        :tags: feature, sql
        :tickets: 3566

        Implemented support for "table valued functions" along with additional
        syntaxes supported by PostgreSQL, one of the most commonly requested
        features. Table valued functions are SQL functions that return lists of
        values or rows, and are prevalent in PostgreSQL in the area of JSON
        functions, where the "table value" is commonly referred towards as the
        "record" datatype. Table valued functions are also supported by Oracle and
        SQL Server.

        Features added include:

        * the :meth:`_functions.FunctionElement.table_valued` modifier that creates a table-like
          selectable object from a SQL function
        * A :class:`_sql.TableValuedAlias` construct that renders a SQL function
          as a named table
        * Support for PostgreSQL's special "derived column" syntax that includes
          column names and sometimes datatypes, such as for the
          ``json_to_recordset`` function, using the
          :meth:`_sql.TableValuedAlias.render_derived` method.
        * Support for PostgreSQL's "WITH ORDINALITY" construct using the
          :paramref:`_functions.FunctionElement.table_valued.with_ordinality` parameter
        * Support for selection FROM a SQL function as column-valued scalar, a
          syntax supported by PostgreSQL and Oracle, via the
          :meth:`_functions.FunctionElement.column_valued` method
        * A way to SELECT a single column from a table-valued expression without
          using a FROM clause via the :meth:`_functions.FunctionElement.scalar_table_valued`
          method.

        .. seealso::

          :ref:`tutorial_functions_table_valued` - in the :ref:`unified_tutorial`

    .. change::
        :tags: bug, asyncio
        :tickets: 5827

        Fixed bug in asyncio connection pool where ``asyncio.TimeoutError`` would
        be raised rather than :class:`.exc.TimeoutError`.  Also repaired the
        :paramref:`_sa.create_engine.pool_timeout` parameter set to zero when using
        the async engine, which previously would ignore the timeout and block
        rather than timing out immediately as is the behavior with regular
        :class:`.QueuePool`.

    .. change::
        :tags: bug, postgresql, asyncio
        :tickets: 5824

        Fixed bug in asyncpg dialect where a failure during a "commit" or less
        likely a "rollback" should cancel the entire transaction; it's no longer
        possible to emit rollback. Previously the connection would continue to
        await a rollback that could not succeed as asyncpg would reject it.

    .. change::
        :tags: bug, orm

        Fixed an issue where the API to create a custom executable SQL construct
        using the ``sqlalchemy.ext.compiles`` extension according to the
        documentation that's been up for many years would no longer function if
        only ``Executable, ClauseElement`` were used as the base classes,
        additional classes were needed if wanting to use
        :meth:`_orm.Session.execute`. This has been resolved so that those extra
        classes aren't needed.

    .. change::
        :tags: bug, regression, orm
        :tickets: 5867

        Fixed ORM unit of work regression where an errant "assert primary_key"
        statement interferes with primary key generation sequences that don't
        actually consider the columns in the table to use a real primary key
        constraint, instead using :paramref:`_orm.mapper.primary_key` to establish
        certain columns as "primary".

    .. change::
        :tags: bug, sql
        :tickets: 5722
        :versions: 1.4.0b2

        Properly render ``cycle=False`` and ``order=False`` as ``NO CYCLE`` and
        ``NO ORDER`` in :class:`_sql.Sequence` and :class:`_sql.Identity`
        objects.

    .. change::
        :tags: schema, usecase
        :tickets: 2843

        Added parameters :paramref:`_ddl.CreateTable.if_not_exists`,
        :paramref:`_ddl.CreateIndex.if_not_exists`,
        :paramref:`_ddl.DropTable.if_exists` and
        :paramref:`_ddl.DropIndex.if_exists` to the :class:`_ddl.CreateTable`,
        :class:`_ddl.DropTable`, :class:`_ddl.CreateIndex` and
        :class:`_ddl.DropIndex` constructs which result in "IF NOT EXISTS" / "IF
        EXISTS" DDL being added to the CREATE/DROP. These phrases are not accepted
        by all databases and the operation will fail on a database that does not
        support it as there is no similarly compatible fallback within the scope of
        a single DDL statement.  Pull request courtesy Ramon Williams.

    .. change::
        :tags: bug, pool, asyncio
        :tickets: 5823

        When using an asyncio engine, the connection pool will now detach and
        discard a pooled connection that is was not explicitly closed/returned to
        the pool when its tracking object is garbage collected, emitting a warning
        that the connection was not properly closed.   As this operation occurs
        during Python gc finalizers, it's not safe to run any IO operations upon
        the connection including transaction rollback or connection close as this
        will often be outside of the event loop.

        The ``AsyncAdaptedQueue`` used by default on async dpapis
        should instantiate a queue only when it's first used
        to avoid binding it to a possibly wrong event loop.

.. changelog::
    :version: 1.4.0b1
    :released: March 15, 2021
    :released: November 2, 2020

    .. change::
        :tags: feature, orm
        :tickets: 5159

        The ORM can now generate queries previously only available when using
        :class:`_orm.Query` using the :func:`_sql.select` construct directly.
        A new system by which ORM "plugins" may establish themselves within a
        Core :class:`_sql.Select` allow the majority of query building logic
        previously inside of :class:`_orm.Query` to now take place within
        a compilation-level extension for :class:`_sql.Select`.  Similar changes
        have been made for the :class:`_sql.Update` and :class:`_sql.Delete`
        constructs as well.  The constructs when invoked using :meth:`_orm.Session.execute`
        now do ORM-related work within the method. For :class:`_sql.Select`,
        the :class:`_engine.Result` object returned now contains ORM-level
        entities and results.

        .. seealso::

            :ref:`change_5159`

    .. change::
        :tags: feature,sql
        :tickets: 4737

        Added "from linting" as a built-in feature to the SQL compiler.  This
        allows the compiler to maintain graph of all the FROM clauses in a
        particular SELECT statement, linked by criteria in either the WHERE
        or in JOIN clauses that link these FROM clauses together.  If any two
        FROM clauses have no path between them, a warning is emitted that the
        query may be producing a cartesian product.   As the Core expression
        language as well as the ORM are built on an "implicit FROMs" model where
        a particular FROM clause is automatically added if any part of the query
        refers to it, it is easy for this to happen inadvertently and it is
        hoped that the new feature helps with this issue.

        .. seealso::

            :ref:`change_4737`

    .. change::
        :tags: deprecated, orm
        :tickets: 5606

        The "slice index" feature used by :class:`_orm.Query` as well as by the
        dynamic relationship loader will no longer accept negative indexes in
        SQLAlchemy 2.0.  These operations do not work efficiently and load the
        entire collection in, which is both surprising and undesirable.   These
        will warn in 1.4 unless the :paramref:`_orm.Session.future` flag is set in
        which case they will raise IndexError.


    .. change::
        :tags: sql, change
        :tickets: 4617

        The "clause coercion" system, which is SQLAlchemy Core's system of receiving
        arguments and resolving them into :class:`_expression.ClauseElement` structures in order
        to build up SQL expression objects, has been rewritten from a series of
        ad-hoc functions to a fully consistent class-based system.   This change
        is internal and should have no impact on end users other than more specific
        error messages when the wrong kind of argument is passed to an expression
        object, however the change is part of a larger set of changes involving
        the role and behavior of :func:`_expression.select` objects.


    .. change::
        :tags: bug, mysql

        The MySQL and MariaDB dialects now query from the information_schema.tables
        system view in order to determine if a particular table exists or not.
        Previously, the "DESCRIBE" command was used with an exception catch to
        detect non-existent,  which would have the undesirable effect of emitting a
        ROLLBACK on the connection. There appeared to be legacy encoding issues
        which prevented the use of "SHOW TABLES", for this, but as MySQL support is
        now at 5.0.2  or above due to :ticket:`4189`, the information_schema tables
        are now available in all cases.


    .. change::
        :tags: bug, orm
        :tickets: 5122

        A query that is against a mapped inheritance subclass which also uses
        :meth:`_query.Query.select_entity_from` or a similar technique in order  to
        provide an existing subquery to SELECT from, will now raise an error if the
        given subquery returns entities that do not correspond to the given
        subclass, that is, they are sibling or superclasses in the same hierarchy.
        Previously, these would be returned without error.  Additionally, if the
        inheritance mapping is a single-inheritance mapping, the given subquery
        must apply the appropriate filtering against the polymorphic discriminator
        column in order to avoid this error; previously, the :class:`_query.Query` would
        add this criteria to the outside query however this interferes with some
        kinds of query that return other kinds of entities as well.

        .. seealso::

            :ref:`change_5122`

    .. change::
        :tags: bug, engine
        :tickets: 5004

        Revised the :paramref:`.Connection.execution_options.schema_translate_map`
        feature such that the processing of the SQL statement to receive a specific
        schema name occurs within the execution phase of the statement, rather than
        at the compile phase.   This is to support the statement being efficiently
        cached.   Previously, the current schema being rendered into the statement
        for a particular run would be considered as part of the cache key itself,
        meaning that for a run against hundreds of schemas, there would be hundreds
        of cache keys, rendering the cache much less performant.  The new behavior
        is that the rendering is done in a similar  manner as the "post compile"
        rendering added in 1.4 as part of :ticket:`4645`, :ticket:`4808`.

    .. change::
        :tags: usecase, sql
        :tickets: 527

        The :meth:`.Index.create` and :meth:`.Index.drop` methods now have a
        parameter :paramref:`.Index.create.checkfirst`, in the same way as that of
        :class:`_schema.Table` and :class:`.Sequence`, which when enabled will cause the
        operation to detect if the index exists (or not) before performing a create
        or drop operation.


    .. change::
        :tags: sql, postgresql
        :tickets: 5498

        Allow specifying the data type when creating a :class:`.Sequence` in
        PostgreSQL by using the parameter :paramref:`.Sequence.data_type`.

    .. change::
        :tags: change, mssql
        :tickets: 5084

        SQL Server OFFSET and FETCH keywords are now used for limit/offset, rather
        than using a window function, for SQL Server versions 11 and higher. TOP is
        still used for a query that features only LIMIT.   Pull request courtesy
        Elkin.

    .. change::
        :tags: deprecated, engine
        :tickets: 5526

        The :class:`_engine.URL` object is now an immutable named tuple. To modify
        a URL object, use the :meth:`_engine.URL.set` method to produce a new URL
        object.

        .. seealso::

            :ref:`change_5526` - notes on migration


    .. change::
        :tags: change, postgresql

        When using the psycopg2 dialect for PostgreSQL, psycopg2 minimum version is
        set at 2.7. The psycopg2 dialect relies upon many features of psycopg2
        released in the past few years, so to simplify the dialect, version 2.7,
        released in March, 2017 is now the minimum version required.


    .. change::
        :tags: usecase, sql

        The :func:`.true` and :func:`.false` operators may now be applied as the
        "onclause" of a :func:`_expression.join` on a backend that does not support
        "native boolean" expressions, e.g. Oracle or SQL Server, and the expression
        will render as "1=1" for true and "1=0" false.  This is the behavior that
        was introduced many years ago in :ticket:`2804` for and/or expressions.

    .. change::
        :tags: feature, engine
        :tickets: 5087, 4395, 4959

        Implemented an all-new :class:`_result.Result` object that replaces the previous
        ``ResultProxy`` object.   As implemented in Core, the subclass
        :class:`_result.CursorResult` features a compatible calling interface with the
        previous ``ResultProxy``, and additionally adds a great amount of new
        functionality that can be applied to Core result sets as well as ORM result
        sets, which are now integrated into the same model.   :class:`_result.Result`
        includes features such as column selection and rearrangement, improved
        fetchmany patterns, uniquing, as well as a variety of implementations that
        can be used to create database results from in-memory structures as well.


        .. seealso::

            :ref:`change_result_14_core`


    .. change::
        :tags: renamed, engine
        :tickets: 5244

        The :meth:`_reflection.Inspector.reflecttable` was renamed to
        :meth:`_reflection.Inspector.reflect_table`.

    .. change::
        :tags: change, orm
        :tickets: 4662

        The condition where a pending object being flushed with an identity that
        already exists in the identity map has been adjusted to emit a warning,
        rather than throw a :class:`.FlushError`. The rationale is so that the
        flush will proceed and raise a :class:`.IntegrityError` instead, in the
        same way as if the existing object were not present in the identity map
        already.   This helps with schemes that are using the
        :class:`.IntegrityError` as a means of catching whether or not a row
        already exists in the table.

        .. seealso::

            :ref:`change_4662`


    .. change::
        :tags: bug, sql
        :tickets: 5001

        Fixed issue where when constructing constraints from ORM-bound columns,
        primarily :class:`_schema.ForeignKey` objects but also :class:`.UniqueConstraint`,
        :class:`.CheckConstraint` and others, the ORM-level
        :class:`.InstrumentedAttribute` is discarded entirely, and all ORM-level
        annotations from the columns are removed; this is so that the constraints
        are still fully pickleable without the ORM-level entities being pulled in.
        These annotations are not necessary to be present at the schema/metadata
        level.

    .. change::
        :tags: bug, mysql
        :tickets: 5568

        The "skip_locked" keyword used with ``with_for_update()`` will render "SKIP
        LOCKED" on all MySQL backends, meaning it will fail for MySQL less than
        version 8 and on current MariaDB backends.  This is because those backends
        do not support "SKIP LOCKED" or any equivalent, so this error should not be
        silently ignored.   This is upgraded from a warning in the 1.3 series.


    .. change::
        :tags: performance, postgresql
        :tickets: 5401

        The psycopg2 dialect now defaults to using the very performant
        ``execute_values()`` psycopg2 extension for compiled INSERT statements,
        and also implements RETURNING support when this extension is used.  This
        allows INSERT statements that even include an autoincremented SERIAL
        or IDENTITY value to run very fast while still being able to return the
        newly generated primary key values.   The ORM will then integrate this
        new feature in a separate change.

        .. seealso::

            :ref:`change_5401` - full list of changes regarding the
            ``executemany_mode`` parameter.


    .. change::
        :tags: feature, orm
        :tickets: 4472

        Added the ability to add arbitrary criteria to the ON clause generated
        by a relationship attribute in a query, which applies to methods such
        as :meth:`_query.Query.join` as well as loader options like
        :func:`_orm.joinedload`.   Additionally, a "global" version of the option
        allows limiting criteria to be applied to particular entities in
        a query globally.

        .. seealso::

            :ref:`loader_option_criteria`

            :ref:`do_orm_execute_global_criteria`

            :func:`_orm.with_loader_criteria`

    .. change::
        :tags: renamed, sql

        :class:`_schema.Table` parameter ``mustexist`` has been renamed
        to :paramref:`_schema.Table.must_exist` and will now warn when used.

    .. change::
        :tags: removed, sql
        :tickets: 4632

        The "threadlocal" execution strategy, deprecated in 1.3, has been
        removed for 1.4, as well as the concept of "engine strategies" and the
        ``Engine.contextual_connect`` method.  The "strategy='mock'" keyword
        argument is still accepted for now with a deprecation warning; use
        :func:`.create_mock_engine` instead for this use case.

        .. seealso::

            :ref:`change_4393_threadlocal` - from the 1.3 migration notes which
            discusses the rationale for deprecation.

    .. change::
        :tags: mssql, postgresql, reflection, schema, usecase
        :tickets: 4458

        Improved support for covering indexes (with INCLUDE columns). Added the
        ability for postgresql to render CREATE INDEX statements with an INCLUDE
        clause from Core. Index reflection also report INCLUDE columns separately
        for both mssql and postgresql (11+).

    .. change::
        :tags: change, platform
        :tickets: 5400

        The ``importlib_metadata`` library is used to scan for setuptools
        entrypoints rather than pkg_resources.   as importlib_metadata is a small
        library that is included as of Python 3.8, the compatibility library is
        installed as a dependency for Python versions older than 3.8.


    .. change::
        :tags: feature, sql, mssql, oracle
        :tickets: 4808

        Added new "post compile parameters" feature.  This feature allows a
        :func:`.bindparam` construct to have its value rendered into the SQL string
        before being passed to the DBAPI driver, but after the compilation step,
        using the "literal render" feature of the compiler.  The immediate
        rationale for this feature is to support LIMIT/OFFSET schemes that don't
        work or perform well as bound parameters handled by the database driver,
        while still allowing for SQLAlchemy SQL constructs to be cacheable in their
        compiled form.     The immediate targets for the new feature are the "TOP
        N" clause used by SQL Server (and Sybase) which does not support a bound
        parameter, as well as the "ROWNUM" and optional "FIRST_ROWS()" schemes used
        by the Oracle dialect, the former of which has been known to perform better
        without bound parameters and the latter of which does not support a bound
        parameter.   The feature builds upon the mechanisms first developed to
        support "expanding" parameters for IN expressions.   As part of this
        feature, the Oracle ``use_binds_for_limits`` feature is turned on
        unconditionally and this flag is now deprecated.

        .. seealso::

            :ref:`change_4808`

    .. change::
        :tags: feature, sql
        :tickets: 1390

        Add support for regular expression on supported backends.
        Two operations have been defined:

        * :meth:`_sql.ColumnOperators.regexp_match` implementing a regular
          expression match like function.
        * :meth:`_sql.ColumnOperators.regexp_replace` implementing a regular
          expression string replace function.

        Supported backends include SQLite, PostgreSQL, MySQL / MariaDB, and Oracle.

        .. seealso::

            :ref:`change_1390`

    .. change::
        :tags: bug, orm
        :tickets: 4696

        The internal attribute symbols NO_VALUE and NEVER_SET have been unified, as
        there was no meaningful difference between these two symbols, other than a
        few codepaths where they were differentiated in subtle and undocumented
        ways, these have been fixed.


    .. change::
        :tags: oracle, bug

        Correctly render :class:`_schema.Sequence` and :class:`_schema.Identity`
        column options ``nominvalue`` and ``nomaxvalue`` as ``NOMAXVALUE` and
        ``NOMINVALUE`` on oracle database.

    .. change::
        :tags: bug, schema
        :tickets: 4262

        Cleaned up the internal ``str()`` for datatypes so that all types produce a
        string representation without any dialect present, including that it works
        for third-party dialect types without that dialect being present.  The
        string representation defaults to being the UPPERCASE name of that type
        with nothing else.


    .. change::
        :tags: deprecated, sql
        :tickets: 5010

        The :meth:`_sql.Join.alias` method is deprecated and will be removed in
        SQLAlchemy 2.0.   An explicit select + subquery, or aliasing of the inner
        tables, should be used instead.


    .. change::
        :tags: bug, orm
        :tickets: 4194

        Fixed bug where a versioning column specified on a mapper against a
        :func:`_expression.select` construct where the version_id_col itself were against the
        underlying table would incur additional loads when accessed, even if the
        value were locally persisted by the flush.  The actual fix is a result of
        the changes in :ticket:`4617`,  by fact that a :func:`_expression.select` object no
        longer has a ``.c`` attribute and therefore does not confuse the mapper
        into thinking there's an unknown column value present.

    .. change::
        :tags: bug, orm
        :tickets: 3858

        An ``UnmappedInstanceError`` is now raised for :class:`.InstrumentedAttribute`
        if an instance is an unmapped object. Prior to this an ``AttributeError``
        was raised. Pull request courtesy Ramon Williams.

    .. change::
        :tags: removed, platform
        :tickets: 5634

        Dropped support for python 3.4 and 3.5 that has reached EOL. SQLAlchemy 1.4
        series requires python 2.7 or 3.6+.

        .. seealso::

            :ref:`change_5634`

    .. change::
        :tags: performance, sql
        :tickets: 4639

        An all-encompassing reorganization and refactoring of Core and ORM
        internals now allows all Core and ORM statements within the areas of
        DQL (e.g. SELECTs) and DML (e.g. INSERT, UPDATE, DELETE) to allow their
        SQL compilation as well as the construction of result-fetching metadata
        to be fully cached in most cases.   This effectively provides a transparent
        and generalized version of what the "Baked Query" extension has offered
        for the ORM in past versions.  The new feature can calculate the
        cache key for any given SQL construction based on the string that
        it would ultimately produce for a given dialect, allowing functions that
        compose the equivalent select(), Query(), insert(), update() or delete()
        object each time to have that statement cached after it's generated
        the first time.

        The feature is enabled transparently but includes some new programming
        paradigms that may be employed to make the caching even more efficient.

        .. seealso::

            :ref:`change_4639`

            :ref:`sql_caching`

    .. change::
        :tags: orm, removed
        :tickets: 4638

        All long-deprecated "extension" classes have been removed, including
        MapperExtension, SessionExtension, PoolListener, ConnectionProxy,
        AttributeExtension.  These classes have been deprecated since version 0.7
        long superseded by the event listener system.


    .. change::
        :tags: feature, mssql, sql
        :tickets: 4384

        Added support for the :class:`_types.JSON` datatype on the SQL Server
        dialect using the :class:`_mssql.JSON` implementation, which implements SQL
        Server's JSON functionality against the ``NVARCHAR(max)`` datatype as per
        SQL Server documentation. Implementation courtesy Gord Thompson.

    .. change::
        :tags: change, sql
        :tickets: 4868

        Added a core :class:`Values` object that enables a VALUES construct
        to be used in the FROM clause of an SQL statement for databases that
        support it (mainly PostgreSQL and SQL Server).

    .. change::
        :tags: usecase, mysql
        :tickets: 5496

        Added a new dialect token "mariadb" that may be used in place of "mysql" in
        the :func:`_sa.create_engine` URL.  This will deliver a MariaDB dialect
        subclass of the MySQLDialect in use that forces the "is_mariadb" flag to
        True.  The dialect will raise an error if a server version string that does
        not indicate MariaDB in use is received.   This is useful for
        MariaDB-specific testing scenarios as well as to support applications that
        are hardcoding to MariaDB-only concepts.  As MariaDB and MySQL featuresets
        and usage patterns continue to diverge, this pattern may become more
        prominent.


    .. change::
        :tags: bug, postgresql

        The pg8000 dialect has been revised and modernized for the most recent
        version of the pg8000 driver for PostgreSQL.  Changes to the dialect
        include:

        * All data types are now sent as text rather than binary.

        * Using adapters, custom types can be plugged in to pg8000.

        * Previously, named prepared statements were used for all statements.
          Now unnamed prepared statements are used by default, and named
          prepared statements can be used explicitly by calling the
          Connection.prepare() method, which returns a PreparedStatement
          object.

        Pull request courtesy Tony Locke.

    .. change::
        :tags: bug, orm
        :tickets: 5074

        The :class:`.Session` object no longer initiates a
        :class:`.SessionTransaction` object immediately upon construction or after
        the previous transaction is closed; instead, "autobegin" logic now
        initiates the new :class:`.SessionTransaction` on demand when it is next
        needed.  Rationale includes to remove reference cycles from a
        :class:`.Session` that has been closed out, as well as to remove the
        overhead incurred by the creation of :class:`.SessionTransaction` objects
        that are often discarded immediately. This change affects the behavior of
        the :meth:`.SessionEvents.after_transaction_create` hook in that the event
        will be emitted when the :class:`.Session` first requires a
        :class:`.SessionTransaction` be present, rather than whenever the
        :class:`.Session` were created or the previous :class:`.SessionTransaction`
        were closed.   Interactions with the :class:`_engine.Engine` and the database
        itself remain unaffected.

        .. seealso::

            :ref:`change_5074`


    .. change::
        :tags: oracle, change

        The LIMIT / OFFSET scheme used in Oracle now makes use of named subqueries
        rather than unnamed subqueries when it transparently rewrites a SELECT
        statement to one that uses a subquery that includes ROWNUM.  The change is
        part of a larger change where unnamed subqueries are no longer directly
        supported by Core, as well as to modernize the internal use of the select()
        construct within the Oracle dialect.


    .. change::
        :tags: feature, engine, orm
        :tickets: 3414

        SQLAlchemy now includes support for Python asyncio within both Core and
        ORM, using the included :ref:`asyncio extension <asyncio_toplevel>`. The
        extension makes use of the `greenlet
        <https://greenlet.readthedocs.io/en/latest/>`_ library in order to adapt
        SQLAlchemy's sync-oriented internals such that an asyncio interface that
        ultimately interacts with an asyncio database adapter is now feasible.  The
        single driver supported at the moment is the
        :ref:`dialect-postgresql-asyncpg` driver for PostgreSQL.

        .. seealso::

            :ref:`change_3414`


    .. change::
        :tags: removed, sql

        Removed the ``sqlalchemy.sql.visitors.iterate_depthfirst`` and
        ``sqlalchemy.sql.visitors.traverse_depthfirst`` functions.  These functions
        were unused by any part of SQLAlchemy.  The
        :func:`_sa.sql.visitors.iterate` and :func:`_sa.sql.visitors.traverse`
        functions are commonly used for these functions.  Also removed unused
        options from the remaining functions including "column_collections",
        "schema_visitor".


    .. change::
        :tags: orm, performance

        The bulk update and delete methods :meth:`.Query.update` and
        :meth:`.Query.delete`, as well as their 2.0-style counterparts, now make
        use of RETURNING when the "fetch" strategy is used in order to fetch the
        list of affected primary key identites, rather than emitting a separate
        SELECT, when the backend in use supports RETURNING.  Additionally, the
        "fetch" strategy will in ordinary cases not expire the attributes that have
        been updated, and will instead apply the updated values directly in the
        same way that the "evaluate" strategy does, to avoid having to refresh the
        object.   The "evaluate" strategy will also fall back to expiring
        attributes that were updated to a SQL expression that was unevaluable in
        Python.

        .. seealso::

            :ref:`change_orm_update_returning_14`

    .. change::
        :tags: bug, orm
        :tickets: 4829

        Added new entity-targeting capabilities to the ORM query context
        help with the case where the :class:`.Session` is using a bind dictionary
        against mapped classes, rather than a single bind, and the :class:`_query.Query`
        is against a Core statement that was ultimately generated from a method
        such as :meth:`_query.Query.subquery`.  First implemented using a deep
        search, the current approach leverages the unified :func:`_sql.select`
        construct to keep track of the first mapper that is part of
        the construct.


    .. change::
        :tags: mssql

        The mssql dialect will assume that at least MSSQL 2005 is used.
        There is no hard exception raised if a previous version is detected,
        but operations may fail for older versions.

    .. change::
        :tags: bug, inheritance, orm
        :tickets: 4212

        An :class:`.ArgumentError` is now raised if both the ``selectable`` and
        ``flat`` parameters are set to True in :func:`.orm.with_polymorphic`. The
        selectable name is already aliased and applying flat=True overrides the
        selectable name with an anonymous name that would've previously caused the
        code to break. Pull request courtesy Ramon Williams.

    .. change::
        :tags: mysql, usecase
        :tickets: 4976

        Added support for use of the :class:`.Sequence` construct with MariaDB 10.3
        and greater, as this is now supported by this database.  The construct
        integrates with the :class:`_schema.Table` object in the same way that it does for
        other databases like PostgreSQL and Oracle; if is present on the integer
        primary key "autoincrement" column, it is used to generate defaults.   For
        backwards compatibility, to support a :class:`_schema.Table` that has a
        :class:`.Sequence` on it to support sequence only databases like Oracle,
        while still not having the sequence fire off for MariaDB, the optional=True
        flag should be set, which indicates the sequence should only be used to
        generate the primary key if the target database offers no other option.

        .. seealso::

            :ref:`change_4976`


    .. change::
        :tags: deprecated, engine
        :tickets: 4634

        The :paramref:`_schema.MetaData.bind` argument as well as the overall
        concept of "bound metadata" is deprecated in SQLAlchemy 1.4 and will be
        removed in SQLAlchemy 2.0.  The parameter as well as related functions now
        emit a :class:`_exc.RemovedIn20Warning` when :ref:`deprecation_20_mode` is
        in use.

        .. seealso::

            :ref:`migration_20_implicit_execution`



    .. change::
        :tags: change, extensions
        :tickets: 5142

        Added new parameter :paramref:`_automap.AutomapBase.prepare.autoload_with`
        which supersedes :paramref:`_automap.AutomapBase.prepare.reflect`
        and :paramref:`_automap.AutomapBase.prepare.engine`.



    .. change::
        :tags: usecase, mssql, postgresql
        :tickets: 4966

        Added support for inspection / reflection of partial indexes / filtered
        indexes, i.e. those which use the ``mssql_where`` or ``postgresql_where``
        parameters, with :class:`_schema.Index`.   The entry is both part of the
        dictionary returned by :meth:`.Inspector.get_indexes` as well as part of a
        reflected :class:`_schema.Index` construct that was reflected.  Pull
        request courtesy Ramon Williams.

    .. change::
        :tags: mssql, feature
        :tickets: 4235, 4633

        Added support for "CREATE SEQUENCE" and full :class:`.Sequence` support for
        Microsoft SQL Server.  This removes the deprecated feature of using
        :class:`.Sequence` objects to manipulate IDENTITY characteristics which
        should now be performed using ``mssql_identity_start`` and
        ``mssql_identity_increment`` as documented at :ref:`mssql_identity`. The
        change includes a new parameter :paramref:`.Sequence.data_type` to
        accommodate SQL Server's choice of datatype, which for that backend
        includes INTEGER, BIGINT, and DECIMAL(n, 0).   The default starting value
        for SQL Server's version of :class:`.Sequence` has been set at 1; this
        default is now emitted within the CREATE SEQUENCE DDL for all backends.

        .. seealso::

            :ref:`change_4235`

    .. change::
        :tags: bug, orm
        :tickets: 4718

        Fixed issue in polymorphic loading internals which would fall back to a
        more expensive, soon-to-be-deprecated form of result column lookup within
        certain unexpiration scenarios in conjunction with the use of
        "with_polymorphic".

    .. change::
        :tags: mssql, reflection
        :tickets: 5527

        As part of the support for reflecting :class:`_schema.Identity` objects,
        the method :meth:`_reflection.Inspector.get_columns` no longer returns
        ``mssql_identity_start`` and ``mssql_identity_increment`` as part of the
        ``dialect_options``. Use the information in the ``identity`` key instead.

    .. change::
        :tags: schema, sql
        :tickets: 5362, 5324, 5360

        Added the :class:`_schema.Identity` construct that can be used to
        configure identity columns rendered with GENERATED { ALWAYS |
        BY DEFAULT } AS IDENTITY. Currently the supported backends are
        PostgreSQL >= 10, Oracle >= 12 and MSSQL (with different syntax
        and a subset of functionalities).

    .. change::
        :tags: change, orm, sql

        A selection of Core and ORM query objects now perform much more of their
        Python computational tasks within the compile step, rather than at
        construction time.  This is to support an upcoming caching model that will
        provide for caching of the compiled statement structure based on a cache
        key that is derived from the statement construct, which itself is expected
        to be newly constructed in Python code each time it is used.    This means
        that the internal state of these objects may not be the same as it used to
        be, as well as that some but not all error raise scenarios for various
        kinds of argument validation will occur within the compilation / execution
        phase, rather than at statement construction time.   See the migration
        notes linked below for complete details.

        .. seealso::

            :ref:`change_deferred_construction`


    .. change::
        :tags: usecase, mssql, reflection
        :tickets: 5506

        Added support for reflection of temporary tables with the SQL Server dialect.
        Table names that are prefixed by a pound sign "#" are now introspected from
        the MSSQL "tempdb" system catalog.

    .. change::
        :tags: firebird, deprecated
        :tickets: 5189

        The Firebird dialect is deprecated, as there is now a 3rd party
        dialect that supports this database.

    .. change::
        :tags: misc, deprecated
        :tickets: 5189

        The Sybase dialect is deprecated.


    .. change::
        :tags: mssql, deprecated
        :tickets: 5189

        The adodbapi and mxODBC dialects are deprecated.


    .. change::
        :tags: mysql, deprecated
        :tickets: 5189

        The OurSQL dialect is deprecated.

    .. change::
        :tags: postgresql, deprecated
        :tickets: 5189

        The pygresql and py-postgresql dialects are deprecated.

    .. change::
       :tags: bug, sql
       :tickets: 4649, 4569

       Registered function names based on :class:`.GenericFunction` are now
       retrieved in a case-insensitive fashion in all cases, removing the
       deprecation logic from 1.3 which temporarily allowed multiple
       :class:`.GenericFunction` objects to exist with differing cases.   A
       :class:`.GenericFunction` that replaces another on the same name whether or
       not it's case sensitive emits a warning before replacing the object.

    .. change::
        :tags: orm, performance, postgresql
        :tickets: 5263

        Implemented support for the psycopg2 ``execute_values()`` extension
        within the ORM flush process via the enhancements to Core made
        in :ticket:`5401`, so that this extension is used
        both as a strategy to batch INSERT statements together as well as
        that RETURNING may now be used among multiple parameter sets to
        retrieve primary key values back in batch.   This allows nearly
        all INSERT statements emitted by the ORM on behalf of PostgreSQL
        to be submitted in batch and also via the ``execute_values()``
        extension which benches at five times faster than plain
        executemany() for this particular backend.

        .. seealso::

            :ref:`change_5263`

    .. change::
        :tags: change, general
        :tickets: 4789

        "python setup.py test" is no longer a test runner, as this is deprecated by
        Pypa.   Please use "tox" with no arguments for a basic test run.


    .. change::
        :tags: usecase, oracle
        :tickets: 4857

        The max_identifier_length for the Oracle dialect is now 128 characters by
        default, unless compatibility version less than 12.2 upon first connect, in
        which case the legacy length of 30 characters is used.  This is a
        continuation of the issue as committed to the 1.3 series which adds max
        identifier length detection upon first connect as well as warns for the
        change in Oracle server.

        .. seealso::

            :ref:`oracle_max_identifier_lengths` - in the Oracle dialect documentation


    .. change::
        :tags: bug, oracle
        :tickets: 4971

        The :class:`_oracle.INTERVAL` class of the Oracle dialect is now correctly
        a subclass of the abstract version of :class:`.Interval` as well as the
        correct "emulated" base class, which allows for correct behavior under both
        native and non-native modes; previously it was only based on
        :class:`.TypeEngine`.


    .. change::
        :tags: bug, orm
        :tickets: 4994

        An error is raised if any persistence-related "cascade" settings are made
        on a :func:`_orm.relationship` that also sets up viewonly=True.   The "cascade"
        settings now default to non-persistence related settings only when viewonly
        is also set.  This is the continuation from :ticket:`4993` where this
        setting was changed to emit a warning in 1.3.

        .. seealso::

            :ref:`change_4994`



    .. change::
        :tags: bug, sql
        :tickets: 5054

        Creating an :func:`.and_` or :func:`.or_` construct with no arguments or
        empty ``*args`` will now emit a deprecation warning, as the SQL produced is
        a no-op (i.e. it renders as a blank string). This behavior is considered to
        be non-intuitive, so for empty or possibly empty :func:`.and_` or
        :func:`.or_` constructs, an appropriate default boolean should be included,
        such as ``and_(True, *args)`` or ``or_(False, *args)``.   As has been the
        case for many major versions of SQLAlchemy, these particular boolean
        values will not render if the ``*args`` portion is non-empty.

    .. change::
        :tags: removed, sql

        Removed the concept of a bound engine from the :class:`.Compiler` object,
        and removed the ``.execute()`` and ``.scalar()`` methods from
        :class:`.Compiler`. These were essentially forgotten methods from over a
        decade ago and had no practical use, and it's not appropriate for the
        :class:`.Compiler` object itself to be maintaining a reference to an
        :class:`_engine.Engine`.

    .. change::
       :tags: performance, engine
       :tickets: 4524

       The pool "pre-ping" feature has been refined to not invoke for a DBAPI
       connection that was just opened in the same checkout operation.  pre ping
       only applies to a DBAPI connection that's been checked into the pool
       and is being checked out again.

    .. change::
        :tags: deprecated, engine

        The ``server_side_cursors`` engine-wide parameter is deprecated and will be
        removed in a future release.  For unbuffered cursors, the
        :paramref:`_engine.Connection.execution_options.stream_results` execution
        option should be used on a per-execution basis.

    .. change::
        :tags: bug, orm
        :tickets: 4699

        Improved declarative inheritance scanning to not get tripped up when the
        same base class appears multiple times in the base inheritance list.


    .. change::
        :tags: orm, change
        :tickets: 4395

        The automatic uniquing of rows on the client side is turned off for the new
        :term:`2.0 style` of ORM querying.  This improves both clarity and
        performance.  However, uniquing of rows on the client side is generally
        necessary when using joined eager loading for collections, as there
        will be duplicates of the primary entity for each element in the
        collection because a join was used.  This uniquing must now be manually
        enabled and can be achieved using the new
        :meth:`_engine.Result.unique` modifier.   To avoid silent failure, the ORM
        explicitly requires the method be called when the result of an ORM
        query in 2.0 style makes use of joined load collections.    The newer
        :func:`_orm.selectinload` strategy is likely preferable for eager loading
        of collections in any case.

        .. seealso::

            :ref:`joinedload_not_uniqued`

    .. change::
        :tags: bug, orm
        :tickets: 4195

        Fixed bug in ORM versioning feature where assignment of an explicit
        version_id for a counter configured against a mapped selectable where
        version_id_col is against the underlying table would fail if the previous
        value were expired; this was due to the fact that the  mapped attribute
        would not be configured with active_history=True.


    .. change::
        :tags: mssql, bug, schema
        :tickets: 5597

        Fixed an issue where :meth:`_reflection.has_table` always returned
        ``False`` for temporary tables.

    .. change::
        :tags: mssql, engine
        :tickets: 4809

        Deprecated the ``legacy_schema_aliasing`` parameter to
        :meth:`_sa.create_engine`.   This is a long-outdated parameter that has
        defaulted to False since version 1.1.

    .. change::
        :tags: usecase, orm
        :tickets: 1653

        The evaluator that takes place within the ORM bulk update and delete for
        synchronize_session="evaluate" now supports the IN and NOT IN operators.
        Tuple IN is also supported.


    .. change::
        :tags: change, sql
        :tickets: 5284

        The :func:`_expression.select` construct is moving towards a new calling
        form that is ``select(col1, col2, col3, ..)``, with all other keyword
        arguments removed, as these are all suited using generative methods.    The
        single list of column or table arguments passed to ``select()`` is still
        accepted, however is no longer necessary if expressions are passed in a
        simple positional style.   Other keyword arguments are disallowed when this
        form is used.


        .. seealso::

            :ref:`change_5284`

    .. change::
        :tags: change, sqlite
        :tickets: 4895

        Dropped support for right-nested join rewriting to support old SQLite
        versions prior to 3.7.16, released in 2013.   It is expected that
        all modern Python versions among those now supported should all include
        much newer versions of SQLite.

        .. seealso::

            :ref:`change_4895`


    .. change::
        :tags: deprecated, engine
        :tickets: 5131

        The :meth:`_engine.Connection.connect` method is deprecated as is the concept of
        "connection branching", which copies a :class:`_engine.Connection` into a new one
        that has a no-op ".close()" method.  This pattern is oriented around the
        "connectionless execution" concept which is also being removed in 2.0.

    .. change::
       :tags: bug, general
       :tickets: 4656, 4689

       Refactored the internal conventions used to cross-import modules that have
       mutual dependencies between them, such that the inspected arguments of
       functions and methods are no longer modified.  This allows tools like
       pylint, Pycharm, other code linters, as well as hypothetical pep-484
       implementations added in the future to function correctly as they no longer
       see missing arguments to function calls.   The new approach is also
       simpler and more performant.

       .. seealso::

            :ref:`change_4656`

    .. change::
        :tags: sql, usecase
        :tickets: 5191

        Change the method ``__str`` of :class:`ColumnCollection` to avoid
        confusing it with a python list of string.

    .. change::
        :tags: sql, reflection
        :tickets: 4741

        The "NO ACTION" keyword for foreign key "ON UPDATE" is now considered to be
        the default cascade for a foreign key on all supporting backends (SQlite,
        MySQL, PostgreSQL) and when detected is not included in the reflection
        dictionary; this is already the behavior for PostgreSQL and MySQL for all
        previous SQLAlchemy versions in any case.   The "RESTRICT" keyword is
        positively stored when detected; PostgreSQL does report on this keyword,
        and MySQL as of version 8.0 does as well.  On earlier MySQL versions, it is
        not reported by the database.

    .. change::
        :tags: sql, reflection
        :tickets: 5527, 5324

        Added support for reflecting "identity" columns, which are now returned
        as part of the structure returned by :meth:`_reflection.Inspector.get_columns`.
        When reflecting full :class:`_schema.Table` objects, identity columns will
        be represented using the :class:`_schema.Identity` construct.
        Currently the supported backends are
        PostgreSQL >= 10, Oracle >= 12 and MSSQL (with different syntax
        and a subset of functionalities).

    .. change::
        :tags: feature, sql
        :tickets: 4753

        The :func:`_expression.select` construct and related constructs now allow for
        duplication of column labels and columns themselves in the columns clause,
        mirroring exactly how column expressions were passed in.   This allows
        the tuples returned by an executed result to match what was SELECTed
        for in the first place, which is how the ORM :class:`_query.Query` works, so
        this establishes better cross-compatibility between the two constructs.
        Additionally, it allows column-positioning-sensitive structures such as
        UNIONs (i.e. :class:`_selectable.CompoundSelect`) to be more intuitively constructed
        in those cases where a particular column might appear in more than one
        place.   To support this change, the :class:`_expression.ColumnCollection` has been
        revised to support duplicate columns as well as to allow integer index
        access.

        .. seealso::

            :ref:`change_4753`


    .. change::
        :tags: renamed, sql
        :tickets: 4617

        The :meth:`_expression.SelectBase.as_scalar` and :meth:`_query.Query.as_scalar` methods have
        been renamed to :meth:`_expression.SelectBase.scalar_subquery` and
        :meth:`_query.Query.scalar_subquery`, respectively.  The old names continue to
        exist within 1.4 series with a deprecation warning.  In addition, the
        implicit coercion of :class:`_expression.SelectBase`, :class:`_expression.Alias`, and other
        SELECT oriented objects into scalar subqueries when evaluated in a column
        context is also deprecated, and emits a warning that the
        :meth:`_expression.SelectBase.scalar_subquery` method should be called explicitly.
        This warning will in a later major release become an error, however the
        message will always be clear when :meth:`_expression.SelectBase.scalar_subquery` needs
        to be invoked.   The latter part of the change is for clarity and to reduce
        the implicit decisionmaking by the query coercion system.   The
        :meth:`.Subquery.as_scalar` method, which was previously
        ``Alias.as_scalar``, is also deprecated; ``.scalar_subquery()`` should be
        invoked directly from ` :func:`_expression.select` object or :class:`_query.Query` object.

        This change is part of the larger change to convert :func:`_expression.select` objects
        to no longer be directly part of the "from clause" class hierarchy, which
        also includes an overhaul of the clause coercion system.


    .. change::
        :tags: bug, mssql
        :tickets: 4980

        Fixed the base class of the :class:`_mssql.DATETIMEOFFSET` datatype to
        be based on the :class:`.DateTime` class hierarchy, as this is a
        datetime-holding datatype.


    .. change::
        :tags: bug, engine
        :tickets: 4712

        The :class:`_engine.Connection` object will now not clear a rolled-back
        transaction  until the outermost transaction is explicitly rolled back.
        This is essentially the same behavior that the ORM :class:`.Session` has
        had for a long time, where an explicit call to ``.rollback()`` on all
        enclosing transactions is required for the transaction to logically clear,
        even though the DBAPI-level transaction has already been rolled back.
        The new behavior helps with situations such as the "ORM rollback test suite"
        pattern where the test suite rolls the transaction back within the ORM
        scope, but the test harness which seeks to control the scope of the
        transaction externally does not expect a new transaction to start
        implicitly.

        .. seealso::

            :ref:`change_4712`


    .. change::
        :tags: deprecated, orm
        :tickets: 4719

        Calling the :meth:`_query.Query.instances` method without passing a
        :class:`.QueryContext` is deprecated.   The original use case for this was
        that a :class:`_query.Query` could yield ORM objects when given only the entities
        to be selected as well as a DBAPI cursor object.  However, for this to work
        correctly there is essential metadata that is passed from a SQLAlchemy
        :class:`_engine.ResultProxy` that is derived from the mapped column expressions,
        which comes originally from the :class:`.QueryContext`.   To retrieve ORM
        results from arbitrary SELECT statements, the :meth:`_query.Query.from_statement`
        method should be used.


    .. change::
        :tags: deprecated, sql

        The :class:`_schema.Table` class now raises a deprecation warning
        when columns with the same name are defined. To replace a column a new
        parameter :paramref:`_schema.Table.append_column.replace_existing` was
        added to the :meth:`_schema.Table.append_column` method.

        The :meth:`_expression.ColumnCollection.contains_column` will now
        raises an error when called with a string, suggesting the caller
        to use ``in`` instead.

    .. change::
        :tags: deprecated, engine
        :tickets: 4878

        The :paramref:`.case_sensitive` flag on :func:`_sa.create_engine` is
        deprecated; this flag was part of the transition of the result row object
        to allow case sensitive column matching as the default, while providing
        backwards compatibility for the former matching method.   All string access
        for a row should be assumed to be case sensitive just like any other Python
        mapping.


    .. change::
        :tags: bug, sql
        :tickets: 5127

        Improved the :func:`_sql.tuple_` construct such that it behaves predictably
        when used in a columns-clause context.  The SQL tuple is not supported as a
        "SELECT" columns clause element on most backends; on those that do
        (PostgreSQL, not surprisingly), the Python DBAPI does not have a "nested
        type" concept so there are still challenges in fetching rows for such an
        object. Use of :func:`_sql.tuple_` in a :func:`_sql.select` or
        :class:`_orm.Query` will now raise a :class:`_exc.CompileError` at the
        point at which the :func:`_sql.tuple_` object is seen as presenting itself
        for fetching rows (i.e., if the tuple is in the columns clause of a
        subquery, no error is raised).  For ORM use,the :class:`_orm.Bundle` object
        is an explicit directive that a series of columns should be returned as a
        sub-tuple per row and is suggested by the error message. Additionally ,the
        tuple will now render with parenthesis in all contexts. Previously, the
        parenthesization would not render in a columns context leading to
        non-defined behavior.

    .. change::
        :tags: usecase, sql
        :tickets: 5576

        Add support to ``FETCH {FIRST | NEXT} [ count ]
        {ROW | ROWS} {ONLY | WITH TIES}`` in the select for the supported
        backends, currently PostgreSQL, Oracle and MSSQL.

    .. change::
        :tags: feature, engine, alchemy2
        :tickets: 4644

        Implemented the :paramref:`_sa.create_engine.future` parameter which
        enables forwards compatibility with SQLAlchemy 2. is used for forwards
        compatibility with SQLAlchemy 2.   This engine features
        always-transactional behavior with autobegin.

        .. seealso::

            :ref:`migration_20_toplevel`

    .. change::
        :tags: usecase, sql
        :tickets: 4449

        Additional logic has been added such that certain SQL expressions which
        typically wrap a single database column will use the name of that column as
        their "anonymous label" name within a SELECT statement, potentially making
        key-based lookups in result tuples more intuitive.   The primary example of
        this is that of a CAST expression, e.g. ``CAST(table.colname AS INTEGER)``,
        which will export its default name as "colname", rather than the usual
        "anon_1" label, that is, ``CAST(table.colname AS INTEGER) AS colname``.
        If the inner expression doesn't have a name, then the previous "anonymous
        label" logic is used.  When using SELECT statements that make use of
        :meth:`_expression.Select.apply_labels`, such as those emitted by the ORM, the
        labeling logic will produce ``<tablename>_<inner column name>`` in the same
        was as if the column were named alone.   The logic applies right now to the
        :func:`.cast` and :func:`.type_coerce` constructs as well as some
        single-element boolean expressions.

        .. seealso::

            :ref:`change_4449`

    .. change::
        :tags: feature, orm
        :tickets: 5508

        The ORM Declarative system is now unified into the ORM itself, with new
        import spaces under ``sqlalchemy.orm`` and new kinds of mappings.  Support
        for decorator-based mappings without using a base class, support for
        classical style-mapper() calls that have access to the declarative class
        registry for relationships, and full integration of Declarative with 3rd
        party class attribute systems like ``dataclasses`` and ``attrs`` is now
        supported.

        .. seealso::

            :ref:`change_5508`

            :ref:`change_5027`

    .. change::
        :tags: removed, platform
        :tickets: 5094

        Removed all dialect code related to support for Jython and zxJDBC. Jython
        has not been supported by SQLAlchemy for many years and it is not expected
        that the current zxJDBC code is at all functional; for the moment it just
        takes up space and adds confusion by showing up in documentation. At the
        moment, it appears that Jython has achieved Python 2.7 support in its
        releases but not Python 3.   If Jython were to be supported again, the form
        it should take is against the Python 3 version of Jython, and the various
        zxJDBC stubs for various backends should be implemented as a third party
        dialect.


    .. change::
        :tags: feature, sql
        :tickets: 5221

        Enhanced the disambiguating labels feature of the
        :func:`_expression.select` construct such that when a select statement
        is used in a subquery, repeated column names from different tables are now
        automatically labeled with a unique label name, without the need to use the
        full "apply_labels()" feature that combines tablename plus column name.
        The disambiguated labels are available as plain string keys in the .c
        collection of the subquery, and most importantly the feature allows an ORM
        :func:`_orm.aliased` construct against the combination of an entity and an
        arbitrary subquery to work correctly, targeting the correct columns despite
        same-named columns in the source tables, without the need for an "apply
        labels" warning.


        .. seealso::

            :ref:`migration_20_query_from_self` - Illustrates the new
            disambiguation feature as part of a strategy to migrate away from the
            :meth:`_query.Query.from_self` method.

    .. change::
        :tags: usecase, postgresql
        :tickets: 5549

        Added support for PostgreSQL "readonly" and "deferrable" flags for all of
        psycopg2, asyncpg and pg8000 dialects.   This takes advantage of a newly
        generalized version of the "isolation level" API to support other kinds of
        session attributes set via execution options that are reliably reset
        when connections are returned to the connection pool.

        .. seealso::

            :ref:`postgresql_readonly_deferrable`

    .. change::
        :tags: mysql, feature
        :tickets: 5459

        Added support for MariaDB Connector/Python to the mysql dialect. Original
        pull request courtesy Georg Richter.

    .. change::
        :tags: usecase, orm
        :tickets: 5171

        Enhanced logic that tracks if relationships will be conflicting with each
        other when they write to the same column to include simple cases of two
        relationships that should have a "backref" between them.   This means that
        if two relationships are not viewonly, are not linked with back_populates
        and are not otherwise in an inheriting sibling/overriding arrangement, and
        will populate the same foreign key column, a warning is emitted at mapper
        configuration time warning that a conflict may arise.  A new parameter
        :paramref:`_orm.relationship.overlaps` is added to suit those very rare cases
        where such an overlapping persistence arrangement may be unavoidable.


    .. change::
        :tags: deprecated, orm
        :tickets: 4705, 5202

        Using strings to represent relationship names in ORM operations such as
        :meth:`_orm.Query.join`, as well as strings for all ORM attribute names
        in loader options like :func:`_orm.selectinload`
        is deprecated and will be removed in SQLAlchemy 2.0.  The class-bound
        attribute should be passed instead.  This provides much better specificity
        to the given method, allows for modifiers such as ``of_type()``, and
        reduces internal complexity.

        Additionally, the ``aliased`` and ``from_joinpoint`` parameters to
        :meth:`_orm.Query.join` are also deprecated.   The :func:`_orm.aliased`
        construct now provides for a great deal of flexibility and capability
        and should be used directly.

        .. seealso::

            :ref:`migration_20_orm_query_join_strings`

            :ref:`migration_20_query_join_options`

    .. change::
        :tags: change, platform
        :tickets: 5404

        Installation has been modernized to use setup.cfg for most package
        metadata.

    .. change::
        :tags: bug, sql, postgresql
        :tickets: 5653

        Improved support for column names that contain percent signs in the string,
        including repaired issues involving anoymous labels that also embedded a
        column name with a percent sign in it, as well as re-established support
        for bound parameter names with percent signs embedded on the psycopg2
        dialect, using a late-escaping process similar to that used by the
        cx_Oracle dialect.


    .. change::
        :tags: orm, deprecated
        :tickets: 5134

        Deprecated logic in :meth:`_query.Query.distinct` that automatically adds
        columns in the ORDER BY clause to the columns clause; this will be removed
        in 2.0.

        .. seealso::

            :ref:`migration_20_query_distinct`

    .. change::
       :tags: orm, removed
       :tickets: 4642

       Remove the deprecated loader options ``joinedload_all``, ``subqueryload_all``,
       ``lazyload_all``, ``selectinload_all``. The normal version with method chaining
       should be used in their place.

    .. change::
        :tags: bug, sql
        :tickets: 4887

        Custom functions that are created as subclasses of
        :class:`.FunctionElement` will now generate an "anonymous label" based on
        the "name" of the function just like any other :class:`.Function` object,
        e.g. ``"SELECT myfunc() AS myfunc_1"``. While SELECT statements no longer
        require labels in order for the result proxy object to function, the ORM
        still targets columns in rows by using objects as mapping keys, which works
        more reliably when the column expressions have distinct names.  In any
        case, the behavior is  now made consistent between functions generated by
        :attr:`.func` and those generated as custom :class:`.FunctionElement`
        objects.


    .. change::
        :tags: usecase, extensions
        :tickets: 4887

        Custom compiler constructs created using the :mod:`sqlalchemy.ext.compiled`
        extension will automatically add contextual information to the compiler
        when a custom construct is interpreted as an element in the columns
        clause of a SELECT statement, such that the custom element will be
        targetable as a key in result row mappings, which is the kind of targeting
        that the ORM uses in order to match column elements into result tuples.

    .. change::
        :tags: engine, bug
        :tickets: 5497

        Adjusted the dialect initialization process such that the
        :meth:`_engine.Dialect.on_connect` is not called a second time
        on the first connection.   The hook is called first, then the
        :meth:`_engine.Dialect.initialize` is called if that connection is the
        first for that dialect, then no more events are called.   This eliminates
        the two calls to the "on_connect" function which can produce very
        difficult debugging situations.

    .. change::
        :tags: feature, engine, pyodbc
        :tickets: 5649

        Reworked the "setinputsizes()" set of dialect hooks to be correctly
        extensible for any arbirary DBAPI, by allowing dialects individual hooks
        that may invoke cursor.setinputsizes() in the appropriate style for that
        DBAPI.   In particular this is intended to support pyodbc's style of usage
        which is fundamentally different from that of cx_Oracle.  Added support
        for pyodbc.


    .. change::
        :tags: deprecated, engine
        :tickets: 4846

        "Implicit autocommit", which is the COMMIT that occurs when a DML or DDL
        statement is emitted on a connection, is deprecated and won't be part of
        SQLAlchemy 2.0.   A 2.0-style warning is emitted when autocommit takes
        effect, so that the calling code may be adjusted to use an explicit
        transaction.

        As part of this change, DDL methods such as
        :meth:`_schema.MetaData.create_all` when used against an
        :class:`_engine.Engine` will run the operation in a BEGIN block if one is
        not started already.

        .. seealso::

            :ref:`deprecation_20_mode`


    .. change::
        :tags: deprecated, orm
        :tickets: 5573

        Passing keyword arguments to methods such as :meth:`_orm.Session.execute`
        to be passed into the :meth:`_orm.Session.get_bind` method is deprecated;
        the new :paramref:`_orm.Session.execute.bind_arguments` dictionary should
        be passed instead.


    .. change::
       :tags: renamed, schema
       :tickets: 5413

       Renamed the :meth:`_schema.Table.tometadata` method to
       :meth:`_schema.Table.to_metadata`.  The previous name remains with a
       deprecation warning.

    .. change::
       :tags: bug, sql
       :tickets: 4336

       Reworked the :meth:`_expression.ClauseElement.compare` methods in terms of a new
       visitor-based approach, and additionally added test coverage ensuring that
       all :class:`_expression.ClauseElement` subclasses can be accurately compared
       against each other in terms of structure.   Structural comparison
       capability is used to a small degree within the ORM currently, however
       it also may form the basis for new caching features.

    .. change::
        :tags: feature, orm
        :tickets: 1763

        Eager loaders, such as joined loading, SELECT IN loading, etc., when
        configured on a mapper or via query options will now be invoked during
        the refresh on an expired object; in the case of selectinload and
        subqueryload, since the additional load is for a single object only,
        the "immediateload" scheme is used in these cases which resembles the
        single-parent query emitted by lazy loading.

        .. seealso::

            :ref:`change_1763`

    .. change::
        :tags: usecase, orm
        :tickets: 5018, 3903

        The ORM bulk update and delete operations, historically available via the
        :meth:`_orm.Query.update` and :meth:`_orm.Query.delete` methods as well as
        via the :class:`_dml.Update` and :class:`_dml.Delete` constructs for
        :term:`2.0 style` execution, will now automatically accommodate for the
        additional WHERE criteria needed for a single-table inheritance
        discriminator in order to limit the statement to rows referring to the
        specific subtype requested.   The new :func:`_orm.with_loader_criteria`
        construct is also supported for with bulk update/delete operations.

    .. change::
       :tags: engine, removed
       :tickets: 4643

       Remove deprecated method ``get_primary_keys`` in the :class:`.Dialect` and
       :class:`_reflection.Inspector` classes. Please refer to the
       :meth:`.Dialect.get_pk_constraint` and :meth:`_reflection.Inspector.get_primary_keys`
       methods.

       Remove deprecated event ``dbapi_error`` and the method
       ``ConnectionEvents.dbapi_error``. Please refer to the
       :meth:`_events.ConnectionEvents.handle_error` event.
       This change also removes the attributes ``ExecutionContext.is_disconnect``
       and ``ExecutionContext.exception``.

    .. change::
       :tags: removed, postgresql
       :tickets: 4643

       Remove support for deprecated engine URLs of the form ``postgres://``;
       this has emitted a warning for many years and projects should be
       using ``postgresql://``.

    .. change::
       :tags: removed, mysql
       :tickets: 4643

       Remove deprecated dialect ``mysql+gaerdbms`` that has been deprecated
       since version 1.0. Use the MySQLdb dialect directly.

       Remove deprecated parameter ``quoting`` from :class:`.mysql.ENUM`
       and :class:`.mysql.SET` in the ``mysql`` dialect. The values passed to the
       enum or the set are quoted by SQLAlchemy when needed automatically.

    .. change::
       :tags: removed, orm
       :tickets: 4643

       Remove deprecated function ``comparable_property``. Please refer to the
       :mod:`~sqlalchemy.ext.hybrid` extension. This also removes the function
       ``comparable_using`` in the declarative extension.

       Remove deprecated function ``compile_mappers``.  Please use
       :func:`.configure_mappers`

       Remove deprecated method ``collection.linker``. Please refer to the
       :meth:`.AttributeEvents.init_collection` and
       :meth:`.AttributeEvents.dispose_collection` event handlers.

       Remove deprecated method ``Session.prune`` and parameter
       ``Session.weak_identity_map``. See the recipe at
       :ref:`session_referencing_behavior` for an event-based approach to
       maintaining strong identity references.
       This change also removes the class ``StrongInstanceDict``.

       Remove deprecated parameter ``mapper.order_by``. Use :meth:`_query.Query.order_by`
       to determine the ordering of a result set.

       Remove deprecated parameter ``Session._enable_transaction_accounting``.

       Remove deprecated parameter ``Session.is_modified.passive``.

    .. change::
       :tags: removed, schema
       :tickets: 4643

       Remove deprecated class ``Binary``. Please use :class:`.LargeBinary`.

    .. change::
       :tags: removed, sql
       :tickets: 4643

       Remove deprecated methods ``Compiled.compile``, ``ClauseElement.__and__`` and
       ``ClauseElement.__or__`` and attribute ``Over.func``.

       Remove deprecated ``FromClause.count`` method. Please use the
       :class:`_functions.count` function available from the
       :attr:`.func` namespace.

    .. change::
       :tags: removed,  sql
       :tickets: 4643

       Remove deprecated parameters ``text.bindparams`` and ``text.typemap``.
       Please refer to the :meth:`_expression.TextClause.bindparams` and
       :meth:`_expression.TextClause.columns` methods.

       Remove deprecated parameter ``Table.useexisting``. Please use
       :paramref:`_schema.Table.extend_existing`.

    .. change::
        :tags: bug, orm
        :tickets: 4836

        An exception is now raised if the ORM loads a row for a polymorphic
        instance that has a primary key but the discriminator column is NULL, as
        discriminator columns should not be null.



    .. change::
        :tags: bug, sql
        :tickets: 4002

        Deprecate usage of ``DISTINCT ON`` in dialect other than PostgreSQL.
        Deprecate old usage of string distinct in MySQL dialect

    .. change::
        :tags: orm, usecase
        :tickets: 5237

        Update :paramref:`_orm.relationship.sync_backref` flag in a relationship
        to make it implicitly ``False`` in ``viewonly=True`` relationships,
        preventing synchronization events.


        .. seealso::

            :ref:`change_5237_14`

    .. change::
        :tags: deprecated, engine
        :tickets: 4877

        Deprecated the behavior by which a :class:`_schema.Column` can be used as the key
        in a result set row lookup, when that :class:`_schema.Column` is not part of the
        SQL selectable that is being selected; that is, it is only matched on name.
        A deprecation warning is now emitted for this case.   Various ORM use
        cases, such as those involving :func:`_expression.text` constructs, have been improved
        so that this fallback logic is avoided in most cases.


    .. change::
        :tags: change, schema
        :tickets: 5367

        The :paramref:`.Enum.create_constraint` and
        :paramref:`.Boolean.create_constraint` parameters now default to False,
        indicating when a so-called "non-native" version of these two datatypes is
        created, a CHECK constraint will not be generated by default.   These CHECK
        constraints present schema-management maintenance complexities that should
        be opted in to, rather than being turned on by default.

        .. seealso::

            :ref:`change_5367`

    .. change::
        :tags: feature, sql
        :tickets: 4645

        The "expanding IN" feature, which generates IN expressions at query
        execution time which are based on the particular parameters associated with
        the statement execution, is now used for all IN expressions made against
        lists of literal values.   This allows IN expressions to be fully cacheable
        independently of the list of values being passed, and also includes support
        for empty lists. For any scenario where the IN expression contains
        non-literal SQL expressions, the old behavior of pre-rendering for each
        position in the IN is maintained. The change also completes support for
        expanding IN with tuples, where previously type-specific bind processors
        weren't taking effect.

        .. seealso::

            :ref:`change_4645`

    .. change::
        :tags: bug, mysql
        :tickets: 4189

        MySQL dialect's server_version_info tuple is now all numeric.  String
        tokens like "MariaDB" are no longer present so that numeric comparison
        works in all cases.  The .is_mariadb flag on the dialect should be
        consulted for whether or not mariadb was detected.   Additionally removed
        structures meant to support extremely old MySQL versions 3.x and 4.x;
        the minimum MySQL version supported is now version 5.0.2.


    .. change::
        :tags: engine, feature
        :tickets: 2056

        Added new reflection method :meth:`.Inspector.get_sequence_names` which
        returns all the sequences defined and :meth:`.Inspector.has_sequence` to
        check if a particular sequence exits.
        Support for this method has been added to the backend that support
        :class:`.Sequence`: PostgreSQL, Oracle and MariaDB >= 10.3.

    .. change::
        :tags: usecase, postgresql
        :tickets: 4914

        The maximum buffer size for the :class:`.BufferedRowResultProxy`, which
        is used by dialects such as PostgreSQL when ``stream_results=True``, can
        now be set to a number greater than 1000 and the buffer will grow to
        that size.  Previously, the buffer would not go beyond 1000 even if the
        value were set larger.   The growth of the buffer is also now based
        on a simple multiplying factor currently set to 5.  Pull request courtesy
        Soumaya Mauthoor.


    .. change::
        :tags: bug, orm
        :tickets: 4519

        Accessing a collection-oriented attribute on a newly created object no
        longer mutates ``__dict__``, but still returns an empty collection as has
        always been the case.   This allows collection-oriented attributes to work
        consistently in comparison to scalar attributes which return ``None``, but
        also don't mutate ``__dict__``.  In order to accommodate for the collection
        being mutated, the same empty collection is returned each time once
        initially created, and when it is mutated (e.g. an item appended, added,
        etc.) it is then moved into ``__dict__``.  This removes the last of
        mutating side-effects on read-only attribute access within the ORM.

        .. seealso::

            :ref:`change_4519`

    .. change::
        :tags: change, sql
        :tickets: 4617

        As part of the SQLAlchemy 2.0 migration project, a conceptual change has
        been made to the role of the :class:`_expression.SelectBase` class hierarchy,
        which is the root of all "SELECT" statement constructs, in that they no
        longer serve directly as FROM clauses, that is, they no longer subclass
        :class:`_expression.FromClause`.  For end users, the change mostly means that any
        placement of a :func:`_expression.select` construct in the FROM clause of another
        :func:`_expression.select` requires first that it be wrapped in a subquery first,
        which historically is through the use of the :meth:`_expression.SelectBase.alias`
        method, and is now also available through the use of
        :meth:`_expression.SelectBase.subquery`.    This was usually a requirement in any
        case since several databases don't accept unnamed SELECT subqueries
        in their FROM clause in any case.

        .. seealso::

            :ref:`change_4617`

    .. change::
        :tags: change, sql
        :tickets: 4617

        Added a new Core class :class:`.Subquery`, which takes the place of
        :class:`_expression.Alias` when creating named subqueries against a :class:`_expression.SelectBase`
        object.   :class:`.Subquery` acts in the same way as :class:`_expression.Alias`
        and is produced from the :meth:`_expression.SelectBase.subquery` method; for
        ease of use and backwards compatibility, the :meth:`_expression.SelectBase.alias`
        method is synonymous with this new method.

        .. seealso::

            :ref:`change_4617`

    .. change::
        :tags: change, orm
        :tickets: 4617

        The ORM will now warn when asked to coerce a :func:`_expression.select` construct into
        a subquery implicitly.  This occurs within places such as the
        :meth:`_query.Query.select_entity_from` and  :meth:`_query.Query.select_from` methods
        as well as within the :func:`.with_polymorphic` function.  When a
        :class:`_expression.SelectBase` (which is what's produced by :func:`_expression.select`) or
        :class:`_query.Query` object is passed directly to these functions and others,
        the ORM is typically coercing them to be a subquery by calling the
        :meth:`_expression.SelectBase.alias` method automatically (which is now superseded by
        the :meth:`_expression.SelectBase.subquery` method).   See the migration notes linked
        below for further details.

        .. seealso::

            :ref:`change_4617`

    .. change::
        :tags: bug, sql
        :tickets: 4617

        The ORDER BY clause of a :class:`_selectable.CompoundSelect`, e.g. UNION, EXCEPT, etc.
        will not render the table name associated with a given column when applying
        :meth:`_selectable.CompoundSelect.order_by` in terms of a :class:`_schema.Table` - bound
        column.   Most databases require that the names in the ORDER BY clause be
        expressed as label names only which are matched to names in the first
        SELECT statement.    The change is related to :ticket:`4617` in that a
        previous workaround was to refer to the ``.c`` attribute of the
        :class:`_selectable.CompoundSelect` in order to get at a column that has no table
        name.  As the subquery is now named, this change allows both the workaround
        to continue to work, as well as allows table-bound columns as well as the
        :attr:`_selectable.CompoundSelect.selected_columns` collections to be usable in the
        :meth:`_selectable.CompoundSelect.order_by` method.

    .. change::
        :tags: bug, orm
        :tickets: 5226

        The refresh of an expired object will now trigger an autoflush if the list
        of expired attributes include one or more attributes that were explicitly
        expired or refreshed using the :meth:`.Session.expire` or
        :meth:`.Session.refresh` methods.   This is an attempt to find a middle
        ground between the normal unexpiry of attributes that can happen in many
        cases where autoflush is not desirable, vs. the case where attributes are
        being explicitly expired or refreshed and it is possible that these
        attributes depend upon other pending state within the session that needs to
        be flushed.   The two methods now also gain a new flag
        :paramref:`.Session.expire.autoflush` and
        :paramref:`.Session.refresh.autoflush`, defaulting to True; when set to
        False, this will disable the autoflush that occurs on unexpire for these
        attributes.

    .. change::
        :tags: feature, sql
        :tickets: 5380

        Along with the new transparent statement caching feature introduced as part
        of :ticket:`4369`, a new feature intended to decrease the Python overhead
        of creating statements is added, allowing lambdas to be used when
        indicating arguments being passed to a statement object such as select(),
        Query(), update(), etc., as well as allowing the construction of full
        statements within lambdas in a similar manner as that of the "baked query"
        system.   The rationale of using lambdas is adapted from that of the "baked
        query" approach which uses lambdas to encapsulate any amount of Python code
        into a callable that only needs to be called when the statement is first
        constructed into a string.  The new feature however is more sophisticated
        in that Python literal values that would be passed as parameters are
        automatically extracted, so that there is no longer a need to use
        bindparam() objects with such queries.   Use of the feature is optional and
        can be used to as small or as great a degree as is desired, while still
        allowing statements to be fully cacheable.

        .. seealso::

            :ref:`engine_lambda_caching`


    .. change::
        :tags: feature, orm
        :tickets: 5027

        Added support for direct mapping of Python classes that are defined using
        the Python ``dataclasses`` decorator.    Pull request courtesy Václav
        Klusák.  The new feature integrates into new support at the Declarative
        level for systems such as ``dataclasses`` and ``attrs``.

        .. seealso::

            :ref:`change_5027`

            :ref:`change_5508`


    .. change::
        :tags: change, engine
        :tickets: 4710

        The ``RowProxy`` class is no longer a "proxy" object, and is instead
        directly populated with the post-processed contents of the DBAPI row tuple
        upon construction.   Now named :class:`.Row`, the mechanics of how the
        Python-level value processors have been simplified, particularly as it impacts the
        format of the C code, so that a DBAPI row is processed into a result tuple
        up front.   The object returned by the :class:`_engine.ResultProxy` is now the
        :class:`.LegacyRow` subclass, which maintains mapping/tuple hybrid behavior,
        however the base :class:`.Row` class now behaves more fully like a named
        tuple.

        .. seealso::

            :ref:`change_4710_core`


    .. change::
        :tags: change, orm
        :tickets: 4710

        The "KeyedTuple" class returned by :class:`_query.Query` is now replaced with the
        Core :class:`.Row` class, which behaves in the same way as KeyedTuple.
        In SQLAlchemy 2.0, both Core and ORM will return result rows using the same
        :class:`.Row` object.   In the interim, Core uses a backwards-compatibility
        class :class:`.LegacyRow` that maintains the former mapping/tuple hybrid
        behavior used by "RowProxy".

        .. seealso::

            :ref:`change_4710_orm`

    .. change::
        :tags: feature, orm
        :tickets: 4826

        Added "raiseload" feature for ORM mapped columns via :paramref:`.orm.defer.raiseload`
        parameter on :func:`.defer` and :func:`.deferred`.   This provides
        similar behavior for column-expression mapped attributes as the
        :func:`.raiseload` option does for relationship mapped attributes.  The
        change also includes some behavioral changes to deferred columns regarding
        expiration; see the migration notes for details.

        .. seealso::

            :ref:`change_4826`


    .. change::
        :tags: bug, orm
        :tickets: 5150

        The behavior of the :paramref:`_orm.relationship.cascade_backrefs` flag
        will be reversed in 2.0 and set to ``False`` unconditionally, such that
        backrefs don't cascade save-update operations from a forwards-assignment to
        a backwards assignment.   A 2.0 deprecation warning is emitted when the
        parameter is left at its default of ``True`` at the point at which such a
        cascade operation actually takes place.   The new behavior can be
        established as always by setting the flag to ``False`` on a specific
        :func:`_orm.relationship`, or more generally can be set up across the board
        by setting the the :paramref:`_orm.Session.future` flag to True.

        .. seealso::

            :ref:`change_5150`

    .. change::
        :tags: deprecated, engine
        :tickets: 4755

        Deprecated remaining engine-level introspection and utility methods
        including :meth:`_engine.Engine.run_callable`, :meth:`_engine.Engine.transaction`,
        :meth:`_engine.Engine.table_names`, :meth:`_engine.Engine.has_table`.   The utility
        methods are superseded by modern context-manager patterns, and the table
        introspection tasks are suited by the :class:`_reflection.Inspector` object.

    .. change::
        :tags: removed, engine
        :tickets: 4755

        The internal dialect method ``Dialect.reflecttable`` has been removed.  A
        review of third party dialects has not found any making use of this method,
        as it was already documented as one that should not be used by external
        dialects.  Additionally, the private ``Engine._run_visitor`` method
        is also removed.


    .. change::
        :tags: removed, engine
        :tickets: 4755

        The long-deprecated ``Inspector.get_table_names.order_by`` parameter has
        been removed.

    .. change::
        :tags: feature, engine
        :tickets: 4755

        The :paramref:`_schema.Table.autoload_with` parameter now accepts an :class:`_reflection.Inspector` object
        directly, as well as any :class:`_engine.Engine` or :class:`_engine.Connection` as was the case before.


    .. change::
        :tags: change, performance, engine, py3k
        :tickets: 5315

        Disabled the "unicode returns" check that runs on dialect startup when
        running under Python 3, which for many years has occurred in order to test
        the current DBAPI's behavior for whether or not it returns Python Unicode
        or Py2K strings for the VARCHAR and NVARCHAR datatypes.  The check still
        occurs by default under Python 2, however the mechanism to test the
        behavior will be removed in SQLAlchemy 2.0 when Python 2 support is also
        removed.

        This logic was very effective when it was needed, however now that Python 3
        is standard, all DBAPIs are expected to return Python 3 strings for
        character datatypes.  In the unlikely case that a third party DBAPI does
        not support this, the conversion logic within :class:`.String` is still
        available and the third party dialect may specify this in its upfront
        dialect flags by setting the dialect level flag ``returns_unicode_strings``
        to one of :attr:`.String.RETURNS_CONDITIONAL` or
        :attr:`.String.RETURNS_BYTES`, both of which will enable Unicode conversion
        even under Python 3.

    .. change::
        :tags: renamed, sql
        :tickets: 5435, 5429

        Several operators are renamed to achieve more consistent naming across
        SQLAlchemy.

        The operator changes are:

        * ``isfalse`` is now ``is_false``
        * ``isnot_distinct_from`` is now ``is_not_distinct_from``
        * ``istrue`` is now ``is_true``
        * ``notbetween`` is now ``not_between``
        * ``notcontains`` is now ``not_contains``
        * ``notendswith`` is now ``not_endswith``
        * ``notilike`` is now ``not_ilike``
        * ``notlike`` is now ``not_like``
        * ``notmatch`` is now ``not_match``
        * ``notstartswith`` is now ``not_startswith``
        * ``nullsfirst`` is now ``nulls_first``
        * ``nullslast`` is now ``nulls_last``
        * ``isnot`` is now ``is_not``
        * ``not_in_`` is now ``not_in``

        Because these are core operators, the internal migration strategy for this
        change is to support legacy terms for an extended period of time -- if not
        indefinitely -- but update all documentation, tutorials, and internal usage
        to the new terms.  The new terms are used to define the functions, and
        the legacy terms have been deprecated into aliases of the new terms.



    .. change::
        :tags: orm, deprecated
        :tickets: 5192

        The :func:`.eagerload` and :func:`.relation` were old aliases and are
        now deprecated. Use :func:`_orm.joinedload` and :func:`_orm.relationship`
        respectively.


    .. change::
        :tags: bug, sql
        :tickets: 4621

        The :class:`_expression.Join` construct no longer considers the "onclause" as a source
        of additional FROM objects to be omitted from the FROM list of an enclosing
        :class:`_expression.Select` object as standalone FROM objects. This applies to an ON
        clause that includes a reference to another  FROM object outside the JOIN;
        while this is usually not correct from a SQL perspective, it's also
        incorrect for it to be omitted, and the behavioral change makes the
        :class:`_expression.Select` / :class:`_expression.Join` behave a bit more intuitively.

