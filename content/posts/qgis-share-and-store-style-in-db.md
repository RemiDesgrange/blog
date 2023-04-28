---
title: "Qgis Share and Store Style in DB"
date: 2022-04-26T23:05:58+02:00
draft: false
tags: ["QGIS", "GIS", "Postgres", "Postgresql", "Database"]
category: ["QGIS", "Database", "Postgresql"]
image: ""
description: "How to store QGIS layer styles in Postgresql and be able to share it"
---

QGIS is able to save your layers "styles" to a postgresql databases. By style, QGIS means _all_ your layer customization (like labels, fields, form etc...).

The annoying part is: by default, the user who created the style is the _only_ one able to open it. 
Actually QGIS store the (postgres) user who created the style in the db, and filter it *QGIS side* [^1].

[^1]: `¯\_(ツ)_/¯` probably for security reason. Hopefully for us, it doesn't select the user via a `SELECT` statement!

So, here is a little hack to make QGIS happy and still being able to open style whatever the user request it.

```sql
BEGIN;
--migrate your existing style to a temp table
CREATE TEMPORARY TABLE save_layer_style(
    id                bigint,
    f_table_catalog   varchar,
    f_table_schema    varchar,
    f_table_name      varchar,
    f_geometry_column varchar,
    stylename         text,
    styleqml          xml,
    stylesld          xml,
    useasdefault      boolean,
    description       text,
    ui                xml,
    owner             text,
    update_time       timestamp,
    type              varchar
);
--copying the style
INSERT INTO save_layer_style SELECT * FROM layer_styles;
DROP TABLE layer_styles;

CREATE TABLE _layer_styles
(
    id                serial primary key,
    f_table_catalog   varchar,
    f_table_schema    varchar,
    f_table_name      varchar,
    f_geometry_column varchar,
    stylename         text,
    styleqml          xml,
    stylesld          xml,
    useasdefault      boolean,
    description       text,
    ui                xml,
    update_time       timestamp default CURRENT_TIMESTAMP,
    type              varchar
);

INSERT INTO _layer_styles SELECT f_table_catalog, f_table_schema, f_table_name, f_geometry_column, stylename, styleqml, stylesld, useasdefault, description, ui, update_time,type FROM save_layer_style;

CREATE VIEW layer_styles AS
SELECT id,
       f_table_catalog,
       f_table_schema,
       f_table_name,
       f_geometry_column,
       stylename,
       styleqml,
       stylesld,
       useasdefault,
       description,
       "current_user"() as owner,
       ui,
       update_time,
       "type"
FROM _layer_styles;

CREATE OR REPLACE FUNCTION f_trigger_insert_update_delete_styles() RETURNS TRIGGER AS
$$
BEGIN
    IF (TG_OP = 'DELETE') THEN
        DELETE FROM _layer_styles WHERE id = OLD.id;
        IF NOT FOUND THEN RETURN NULL; END IF;
    ELSIF (TG_OP = 'UPDATE') THEN
        UPDATE _layer_styles
        SET f_table_catalog=NEW.f_table_catalog,
            f_table_schema=NEW.f_table_schema,
            f_table_name=NEW.f_table_name,
            f_geometry_column=NEW.f_geometry_column,
            stylename=NEW.stylename,
            styleqml=NEW.styleqml,
            stylesld=NEW.stylesld,
            useasdefault=NEW.useasdefault,
            description=NEW.description,
            ui=NEW.ui,
            update_time=NEW.update_time,
            type=NEW.type
        WHERE id = OLD.id;
        RETURN NEW;
    ELSIF (TG_OP = 'INSERT') THEN
        INSERT INTO _layer_styles(f_table_catalog,
                                  f_table_schema,
                                  f_table_name,
                                  f_geometry_column,
                                  styleName,
                                  styleQML,
                                  styleSLD,
                                  useAsDefault,
                                  description,
                                  type)
        VALUES (NEW.f_table_catalog,
                NEW.f_table_schema,
                NEW.f_table_name,
                NEW.f_geometry_column,
                NEW.styleName,
                NEW.styleQML,
                NEW.styleSLD,
                NEW.useAsDefault,
                NEW.description,
                NEW.type);
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER trg_layer_styles
    INSTEAD OF INSERT OR UPDATE OR DELETE
    ON layer_styles
    FOR EACH ROW
EXECUTE FUNCTION f_trigger_insert_update_delete_styles();
COMMIT;
```


This is pretty standard SQL stuff, but it took me maybe an hour of trial and error with qgis to find the right knob.
