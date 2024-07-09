# Geometric-models-
import pandas as pd
import re
from sqlalchemy import create_engine
import urllib



#połączenie z bazą danych Ms SQL Server
params = urllib.parse.quote_plus(r"DRIVER={ODBC Driver 17 for SQL Server};"
                                 r"SERVER=LENOVO;"
                                 r"DATABASE=DBSTEP;"
                                 r"Trusted_Connection=yes;")
engine = create_engine("mssql+pyodbc:///?odbc_connect=%s" % params)


# Ścieżka do pliku STEP
nazwa_pliku = r'D:\STUDIA\PRACA_INŻYNIERSKA\Modele STEP\Walek01_AP214.STEP'

patterns = {
    # Wzorce
    'AXIS2_PLACEMENT_3D': (re.compile(r"#(\d+)=AXIS2_PLACEMENT_3D\('(.*?)',(#\d*|\$|),(#\d*|\$|),(#\d*|\$|)\) ;"), ['EIN', 'name', 'location', 'axis', 'ref_direction']),
    'CARTESIAN_POINT': (re.compile(r"#(\d+)=CARTESIAN_POINT\('(.+?)',\((.+?)\)\) ;"), ['EIN', 'name', 'coordinates']),
    'VECTOR': (re.compile(r"#(\d+)=VECTOR\('(.+?)',(#\d*|\$|),(.+?)\) ;"), ['EIN', 'name', 'orientation', 'magnitude']),
    'DIRECTION': (re.compile(r"#(\d+)=DIRECTION\('(.+?)',\((.+?)\)\) ;"), ['EIN', 'name', 'direction_ratios']),
    'CIRCLE': (re.compile(r"#(\d+)=CIRCLE\('(.+?)',(#\d*|\$|),(.+?)\) ;"), ['EIN', 'name', 'position', 'radius']),
    'ADVANCED_BREP_SHAPE_REPRESENTATION': (re.compile(r"#(\d+)\s*=\s*ADVANCED_BREP_SHAPE_REPRESENTATION\s*\(\s*'([^']+?)'\s*,\s*\(([^)]+?)\)\s*,\s*(#\d*|\$)\s*\)\s*;"), ['EIN', 'name', 'items', 'context_of_items']),
    'MANIFOLD_SOLID_BREP': (re.compile(r"#(\d+)=MANIFOLD_SOLID_BREP\('(.+?)',(#\d*|\$|)\) ;"), ['EIN', 'name', 'outer']),
    'CLOSED_SHELL': (re.compile(r"#(\d+)=CLOSED_SHELL\('(.+?)',\((.+?)\)\) ;"), ['EIN', 'name', 'cfs_faces']),
    'ADVANCED_FACE': (re.compile(r"#(\d+)=ADVANCED_FACE\('(.+?)',\((.+?)\),(#\d*|\$|),(\.\w+\.)\) ;"), ['EIN', 'name', 'bounds', 'face_geometry', 'same_sense']),
    'FACE_OUTER_BOUND': (re.compile(r"#(\d+)=FACE_OUTER_BOUND\('([^']*)',([^,]*),(\.[TF]\.)\) ;"), ['EIN', 'name', 'bound', 'orientation']),
    'EDGE_LOOP': (re.compile(r"#(\d+)=EDGE_LOOP\('([^']*)',\((#\d+(,#\d+)*)\)\) ;"), ['EIN', 'name', 'edge_list']),
    'ORIENTED_EDGE': (re.compile(r"#(\d+)=ORIENTED_EDGE\('([^']*)',([^,]*),([^,]*),([^,]*),(\.[TF]\.)\) ;"), ['EIN', 'name', 'edge_start', 'edge_end', 'edge_element', 'orientation']),
    'EDGE_CURVE': (re.compile(r"#(\d+)=EDGE_CURVE\('([^']*)',(#\d*|\$|)\,(#\d*|\$|)\,(#\d*|\$|)\,(\.[TF]\.)\) ;"), ['EIN', 'name', 'edge_start', 'edge_end', 'edge_geometry', 'same_sense']),
    'LINE': (re.compile(r"#(\d+)=LINE\('(.+?)',(#\d*|\$|),(#\d*|\$|)\) ;"), ['EIN', 'name', 'pnt', 'dir']),
    'VERTEX_POINT': (re.compile(r"#(\d+)=VERTEX_POINT\('([^']*)',(#\d*|\$)\) ;"), ['EIN', 'name', 'vertex_geometry']),
    'PLANE': (re.compile(r"#(\d+)=PLANE\('(.+?)',(#\d*|\$|)\) ;"), ['EIN', 'name', 'position']),
    'CYLINDRICAL_SURFACE': (re.compile(r"#(\d+)=CYLINDRICAL_SURFACE\('(.+?)',(#\d*|\$|),(\d+\.\d*)\) ;"), ['EIN', 'name', 'position', 'radius']),
    'CONICAL_SURFACE': (re.compile(r"#(\d+)=CONICAL_SURFACE\('(.+?)',(#\d*|\$|),(\d+\.\d*),(\d+\.\d*)\) ;"), ['EIN', 'name', 'position', 'radius', 'semi_angle']),
    'SPHERICAL_SURFACE': (re.compile(r"#(\d+)=SPHERICAL_SURFACE\('(.+?)',(#\d*|\$|),(\d+\.\d*)\) ;"), ['EIN', 'name', 'position', 'radius']),
    'TOROIDAL_SURFACE': (re.compile(r"#(\d+)=TOROIDAL_SURFACE\('(.+?)',(#\d*|\$|),(\d+\.\d*),(\d+\.\d*)\) ;"), ['EIN', 'name', 'position', 'major_radius', 'minor_radius']),
    'B_SPLINE_CURVE_WITH_KNOTS': (re.compile(r"#(\d+)=B_SPLINE_CURVE_WITH_KNOTS\('([^']*)',(\d+),\((#\d+(?:,#\d+)*)\),(\.UNSPECIFIED\.|\.[A-Z_]+\.),(\.T\.|\.F\.|\.U\.),(\.T\.|\.F\.|\.U\.),\((\d+(?:,\d+)*)\),\(([\d., ]+)\),(\.UNSPECIFIED\.|\.UNIFORM_KNOTS\.|\.QUASI_UNIFORM_KNOTS\.|\.PIECEWISE_BEZIER_KNOTS\.)\)"), ['EIN', 'name', 'degree', 'control_points_list', 'curve_form', 'closed_curve', 'self_intersect', 'knot_multiplicities', 'knots', 'knot_spec']),
    'FACE_BOUND': (re.compile(r"#(\d+)=FACE_BOUND\('([^']*)',(#\d*|\$|),(\.[TF]\.)\) ;"), ['EIN', 'name', 'bound', 'orientation']),
    'PRESENTATION_LAYER_ASSIGNMENT': (re.compile(r"#(\d+)=PRESENTATION_LAYER_ASSIGNMENT\('(.+?)',(.+?),\((.+?)\)\) ;"), ['EIN', 'name', 'description', 'assigned_items']),
    'STYLED_ITEM': (re.compile(r"#(\d+)=STYLED_ITEM\('(.+?)',\((.+?)\),(#\d*|\$|)\) ;"), ['EIN', 'name', 'styles', 'item']),
    'PRESENTATION_STYLE_ASSIGNMENT': (re.compile(r"#(\d+)=PRESENTATION_STYLE_ASSIGNMENT\((.+?)\) ;"), ['EIN', 'styles']),
    'SURFACE_STYLE_USAGE': (re.compile(r"#(\d+)=SURFACE_STYLE_USAGE\((.+?)\) ;"), ['EIN', 'side', 'style']),
    'SURFACE_SIDE_STYLE': (re.compile(r"#(\d+)=SURFACE_SIDE_STYLE\('(.+?)',\((.+?)\)\) ;"), ['EIN', 'name', 'styles']),
    'SURFACE_STYLE_FILL_AREA': (re.compile(r"#(\d+)=SURFACE_STYLE_FILL_AREA\((.+?)\) ;"), ['EIN', 'fill_area']),
    'FILL_AREA_STYLE': (re.compile(r"#(\d+)=FILL_AREA_STYLE\('(.+?)',\((.+?)\)\) ;"), ['EIN', 'name', 'fill_styles']),
    'FILL_AREA_STYLE_COLOUR': (re.compile(r"#(\d+)=FILL_AREA_STYLE_COLOUR\('(.+?)',(#\d*|\$|)\) ;"), ['EIN', 'name', 'fill_colour']),
    'COLOUR_RGB': (re.compile(r"#(\d+)=COLOUR_RGB\('(.+?)',(\d+\.\d*),(\d+\.\d*),(\d+\.\d*)\) ;"), ['EIN', 'name', 'red', 'green', 'blue']),
    'SHAPE_DEFINITION_REPRESENTATION': (re.compile(r"#(\d+)=SHAPE_DEFINITION_REPRESENTATION\((#\d*|\$|),(#\d*|\$|)\) ;"), ['EIN', 'definition', 'used_representation']),
    'PRODUCT_DEFINITION_SHAPE': (re.compile(r"#(\d+)=PRODUCT_DEFINITION_SHAPE\('(.+?)','(.+?)',(#\d*|\$|)\) ;"), ['EIN', 'name', 'description', 'definition']),
    'PRODUCT_DEFINITION': (re.compile(r"#(\d+)=PRODUCT_DEFINITION\(\s*'([^']*)'\s*,\s*'([^']*)'\s*,\s*(#\d*|\$)\s*,\s*(#\d*|\$)\s*\)\s*;"), ['EIN', 'id', 'description', 'formation', 'frame_of_reference']),
    'PRODUCT_DEFINITION_FORMATION_WITH_SPECIFIED_SOURCE': (re.compile(r"#(\d+)=PRODUCT_DEFINITION_FORMATION_WITH_SPECIFIED_SOURCE\("r"'([^']*)',\s*'([^']*)',(#\d*|\$),\.(.+?)\.\)"r"\s*;"), ['EIN', 'id', 'description', 'of_product', 'make_or_buy']),
    'PRODUCT': (re.compile(r"#(\d+)=PRODUCT\('\s*([^']*)',\s*'([^']*)',\s*'([^']*)'\s*,\((#\d+(?:,#\d+)*)\)\s*\) ;"), ['EIN', 'name', 'id', 'description', 'frame_of_reference']),
    'APPLICATION_CONTEXT': (re.compile(r"#(\d+)=APPLICATION_CONTEXT\('(.+?)'\) ;"), ['EIN', 'application']),
    'APPLICATION_PROTOCOL_DEFINITION': (re.compile(r"#(\d+)=APPLICATION_PROTOCOL_DEFINITION\('([^']+?)','([^']+?)',(\d+),([^,]+)\) ;"),['EIN', 'status', 'application_interpreted_model_schema_name', 'application_protocol_year', 'application']),
    'PRODUCT_CONTEXT': (re.compile(r"#(\d+)=MECHANICAL_CONTEXT\('([^']*)',\s*(#\d+|\$),'([^']*)'\) ;"), ['EIN', 'name', 'frame_of_reference', 'discipline_type']),
    'PRODUCT_DEFINITION_CONTEXT': (re.compile(r"#(\d+)=DESIGN_CONTEXT\('\s*([^']*)',\s*(#\d+|\$),\s*'([^']*)'\)\s*;"), ['EIN', 'frame_of_reference', 'life_cycle_stage']),
    'SHAPE_REPRESENTATION': (re.compile(r"#(\d+)=SHAPE_REPRESENTATION\('(.+?)',\((.+?)\),(#\d*|\$|)\) ;"), ['EIN', 'name', 'items', 'context_of_items']),
}

patternsAP214 = {
    # Wzorce
    'AXIS2_PLACEMENT_3D': (re.compile(r"#(\d+) = AXIS2_PLACEMENT_3D \(\s*'([^']*)',\s*(#\d+|\$),\s*(#\d+|\$),\s*(#\d+|\$)\s*\)\s*;"), ['EIN', 'name', 'location', 'axis', 'ref_direction']),
    'CARTESIAN_POINT': (re.compile(r"#(\d+) = CARTESIAN_POINT \(\s*'([^']*)',\s*\(\s*(.+?)\s*\)\s*\) ;"), ['EIN', 'name', 'coordinates']),
    'VECTOR': (re.compile(r"#(\d+)\s*=\s*VECTOR\s*\(\s*'([^']*)',\s*(#\d+|\$),\s*([0-9.]+)\s*\)\s*;"), ['EIN', 'name', 'orientation', 'magnitude']),
    'DIRECTION': (re.compile(r"#(\d+) = DIRECTION \(\s*'([^']*)',\s*\(\s*(.+?)\s*\)\s*\) ;"), ['EIN', 'name', 'direction_ratios']),
    'CIRCLE': (re.compile(r"#(\d+) = CIRCLE \(\s*'([^']*)',\s*(#\d+|\$),\s*(.+?)\s*\) ;"), ['EIN', 'name', 'position', 'radius']),
    'ADVANCED_BREP_SHAPE_REPRESENTATION': (re.compile(r"#(\d+) = ADVANCED_BREP_SHAPE_REPRESENTATION \(\s*'([^']*)',\s*\(\s*(#\d+(?:,\s*#\d+)*)\s*\)\s*,\s*(#\d+|\$)\s*\) ;"), ['EIN', 'name', 'items', 'context_of_items']),
    'MANIFOLD_SOLID_BREP': (re.compile(r"#(\d+) = MANIFOLD_SOLID_BREP \(\s*'([^']*)',\s*(#\d+|\$)\s*\) ;"), ['EIN', 'name', 'outer']),
    'CLOSED_SHELL': (re.compile(r"#(\d+) = CLOSED_SHELL \(\s*'([^']*)',\s*\(\s*(#\d+(,\s*#\d+)*)\s*\)\s*\) ;"), ['EIN', 'name', 'cfs_faces']),
    'ADVANCED_FACE': (re.compile(r"#(\d+)\s*=\s*ADVANCED_FACE\s*\(\s*'([^']*)',\s*\(\s*(#\d+(?:,\s*#\d+)*)\s*\),\s*(#\d+|\$),\s*(\.\w+\.)\s*\)\s*;"), ['EIN', 'name', 'bounds', 'face_geometry', 'same_sense']),
    'FACE_OUTER_BOUND': (re.compile(r"#(\d+) = FACE_OUTER_BOUND \(\s*'([^']*)',\s*(#\d+|\$),\s*(\.\w+\.)\s*\) ;"), ['EIN', 'name', 'bound', 'orientation']),
    'EDGE_LOOP': (re.compile(r"#(\d+)\s*=\s*EDGE_LOOP\s*\(\s*'([^']*)',\s*\(\s*((#\d+)(\s*,\s*#\d+)*)\s*\)\s*\)\s*;"), ['EIN', 'name', 'edge_list']),
    'ORIENTED_EDGE': (re.compile(r"#(\d+) = ORIENTED_EDGE \(\s*'([^']*)',\s*(\*|#\d+|\$),\s*(\*|#\d+|\$),\s*(#\d+|\$),\s*(\.\w+\.)\s*\) ;"), ['EIN', 'name', 'edge_start', 'edge_end', 'edge_element', 'orientation']),
    'EDGE_CURVE': (re.compile(r"#(\d+) = EDGE_CURVE \(\s*'([^']*)',\s*(#\d+|\$),\s*(#\d+|\$),\s*(#\d+|\$),\s*(\.\w+\.)\s*\) ;"), ['EIN', 'name', 'edge_start', 'edge_end', 'edge_geometry', 'same_sense']),
    'LINE': (re.compile(r"#(\d+) = LINE \(\s*'([^']*)',\s*(#\d+|\$),\s*(#\d+|\$)\s*\) ;"), ['EIN', 'name', 'pnt', 'dir']),
    'VERTEX_POINT': (re.compile(r"#(\d+) = VERTEX_POINT \(\s*'([^']*)',\s*(#\d+|\$)\s*\) ;"), ['EIN', 'name', 'vertex_geometry']),
    'PLANE': (re.compile(r"#(\d+) = PLANE \(\s*'([^']*)',\s*(#\d+|\$)\s*\) ;"), ['EIN', 'name', 'position']),
    'CYLINDRICAL_SURFACE': (re.compile(r"#(\d+) = CYLINDRICAL_SURFACE \(\s*'([^']*)',\s*(#\d+|\$),\s*(\d+\.\d+|\d+)\s*\) ;"), ['EIN', 'name', 'position', 'radius']),
    'CONICAL_SURFACE': (re.compile(r"#(\d+) = CONICAL_SURFACE \(\s*'([^']*)',\s*(#\d+|\$),\s*(\d+\.\d+|\d+),\s*(\d+\.\d+|\d+)\s*\) ;"), ['EIN', 'name', 'position', 'radius', 'semi_angle']),
    'SPHERICAL_SURFACE': (re.compile(r"#(\d+)=SPHERICAL_SURFACE\('(.+?)',(#\d*|\$|),(\d+\.\d*)\) ;"), ['EIN', 'name', 'position', 'radius']),
    'TOROIDAL_SURFACE': (re.compile(r"#(\d+)=TOROIDAL_SURFACE\('(.+?)',(#\d*|\$|),(\d+\.\d*),(\d+\.\d*)\) ;"), ['EIN', 'name', 'position', 'major_radius', 'minor_radius']),
    'B_SPLINE_CURVE_WITH_KNOTS': (re.compile(r"#(\d+) = B_SPLINE_CURVE_WITH_KNOTS\s*\(\s*'([^']*)',\s*([0-9]+),\s*\(\s*((?:#\d+\s*,?\s*(?:\n|\r\n?)?)+)\s*\),\s*(\.\w+\.),\s*(\.\w+\.),\s*(\.\w+\.),\s*\(\s*((?:\d+\s*,?\s*(?:\n|\r\n?)?)+)\s*\),\s*\(\s*((?:-?\d+\.\d+\s*,?\s*(?:\n|\r\n?)?)+)\s*\),\s*(\.\w+\.)\s*\)\s*;"),['EIN', 'name', 'degree','control_points_list','curve_form', 'closed_curve', 'self_intersect', 'knot_multiplicities', 'knots', 'knot_spec']),
    'FACE_BOUND': (re.compile(r"#(\d+) = FACE_BOUND \(\s*'([^']*)',\s*(#\d+|\$),\s*(\.\w+\.)\s*\) ;"), ['EIN', 'name', 'bound', 'orientation']),
    'PRESENTATION_LAYER_ASSIGNMENT': (re.compile(r"#(\d+) = PRESENTATION_LAYER_ASSIGNMENT \(\s*'([^']*)',\s*'([^']*)',\s*\(\s*(#\d+(,\s*#\d+)*)\s*\)\s*\) ;"), ['EIN', 'name', 'description', 'assigned_items']),
    'STYLED_ITEM': (re.compile(r"#(\d+) = STYLED_ITEM \(\s*'([^']*)',\s*\(\s*(#\d+)\s*\),\s*(#\d+)\s*\) ;"), ['EIN', 'name', 'styles', 'item']),
    'PRESENTATION_STYLE_ASSIGNMENT': (re.compile(r"#(\d+) = PRESENTATION_STYLE_ASSIGNMENT \(\(\s*(#\d+)\s*\)\s*\)\s*;"), ['EIN', 'styles']),
    'SURFACE_STYLE_USAGE': (re.compile(r"#(\d+) = SURFACE_STYLE_USAGE \(\s*(\.\w+\.)\s*,\s*(#\d+|\$)\s*\) ;"), ['EIN', 'side', 'style']),
    'SURFACE_SIDE_STYLE': (re.compile(r"#(\d+) = SURFACE_SIDE_STYLE \(\s*'([^']*)',\s*\(\s*(#\d+(,\s*#\d+)*)\s*\)\s*\) ;"), ['EIN', 'name', 'styles']),
    'SURFACE_STYLE_FILL_AREA': (re.compile(r"#(\d+) = SURFACE_STYLE_FILL_AREA \(\s*(#\d+|\$)\s*\) ;"), ['EIN', 'fill_area']),
    'FILL_AREA_STYLE': (re.compile(r"#(\d+) = FILL_AREA_STYLE \(\s*'([^']*)',\s*\(\s*(#\d+(,\s*#\d+)*)\s*\)\s*\) ;"), ['EIN', 'name', 'fill_styles']),
    'FILL_AREA_STYLE_COLOUR': (re.compile(r"#(\d+) = FILL_AREA_STYLE_COLOUR \(\s*'([^']*)',\s*(#\d+|\$)\s*\) ;"), ['EIN', 'name', 'fill_colour']),
    'COLOUR_RGB': (re.compile(r"#(\d+) = COLOUR_RGB \(\s*'([^']*)',\s*(\d+\.\d*),\s*(\d+\.\d*),\s*(\d+\.\d*)\s*\) ;"), ['EIN', 'name', 'red', 'green', 'blue']),
    'SHAPE_DEFINITION_REPRESENTATION': (re.compile(r"#(\d+) = SHAPE_DEFINITION_REPRESENTATION \(\s*(#\d+|\$),\s*(#\d+|\$)\s*\) ;"), ['EIN', 'definition', 'used_representation']),
    'PRODUCT_DEFINITION_SHAPE': (re.compile(r"#(\d+) = PRODUCT_DEFINITION_SHAPE \(\s*'([^']*)',\s*'([^']*)',\s*(#\d+|\$)\s*\) ;"), ['EIN', 'name', 'description', 'definition']),
    'PRODUCT_DEFINITION': (re.compile(r"#(\d+) = PRODUCT_DEFINITION \(\s*'([^']*)',\s*'([^']*)',\s*(#\d+|\$),\s*(#\d+|\$)\s*\) ;"), ['EIN', 'id', 'description', 'formation', 'frame_of_reference']),
    'PRODUCT_DEFINITION_FORMATION_WITH_SPECIFIED_SOURCE': (re.compile(r"#(\d+) = PRODUCT_DEFINITION_FORMATION_WITH_SPECIFIED_SOURCE \(\s*'([^']*)',\s*'([^']*)'?,\s*(#\d+|\$),\s*\.(NOT_KNOWN|MAKE_FROM|BUY|MAKE)\.\s*\) ;"), ['EIN', 'id', 'description', 'of_product', 'make_or_buy']),
    'PRODUCT': (re.compile(r"#(\d+) = PRODUCT \(\s*'([^']*)',\s*'([^']*)',\s*('[^']*'|),\s*\(\s*(#\d+)\s*\)\s*\) ;"), ['EIN', 'id', 'name', 'description', 'frame_of_reference']),
    'PRODUCT_CONTEXT': (re.compile(r"#(\d+)\s*=\s*PRODUCT_CONTEXT\s*\(\s*'([^']+)'\s*,\s*(#\d+)\s*,\s*'([^']+)'\s*\)\s*;"), ['EIN', 'name', 'frame_of_reference', 'discipline_type']),
    'APPLICATION_CONTEXT': (re.compile(r"#(\d+) = APPLICATION_CONTEXT \(\s*'([^']+)'\s*\) ;"), ['EIN', 'application']),
    'PRODUCT_DEFINITION_CONTEXT': (re.compile(r"#(\d+) = PRODUCT_DEFINITION_CONTEXT \(\s*'([^']*)',\s*(#\d+),\s*'([^']*)'\s*\) ;"), ['EIN', 'name', 'frame_of_reference', 'life_cycle_stage']),
    'APPLICATION_PROTOCOL_DEFINITION': (re.compile(r"#(\d+) = APPLICATION_PROTOCOL_DEFINITION \(\s*'([^']*)',\s*'([^']*)',\s*(\d+),\s*(#\d+)\s*\) ;"), ['EIN', 'status', 'application_interpreted_model_schema_name', 'application_protocol_year', 'application']),
}


# wartości domyślne dla kolumn, które będą używane w przetwarzaniu danych
default_values = {
    'name': 'Underfined',
    'axis': 'Underfined',
    'ref_direction': 'Underfined',
    'id': 'Unknown',
    'description': 'Unknown',
}


identifiers = {
    'AXIS2_PLACEMENT_3D': 'IDAP',
    'CARTESIAN_POINT': 'IDCP',
    'VECTOR': 'IDVC',
    'DIRECTION': 'IDDR',
    'CIRCLE': 'IDCR',
    'ADVANCED_BREP_SHAPE_REPRESENTATION': 'IDABSR',
    'SHAPE_REPRESENTATION': 'IDSR',
    'MANIFOLD_SOLID_BREP': 'IDMSB',
    'CLOSED_SHELL':'IDCSH',
    'ADVANCED_FACE':'IDAF',
    'FACE_OUTER_BOUND':'IDFOB',
    'EDGE_LOOP': 'IDEL',
    'ORIENTED_EDGE':'IDOE',
    'EDGE_CURVE':'IDEC',
    'LINE':'IDLN',
    'VERTEX_POINT':'IDVP',
    'PLANE':'IDPL',
    'CYLINDRICAL_SURFACE':'IDCYSF',
    'CONICAL_SURFACE':'IDCOSF',
    'SPHERICAL_SURFACE':'IDSPSF',
    'TOROIDAL_SURFACE':'IDTOSF',
    'B_SPLINE_CURVE_WITH_KNOTS':'IDBSCK',
    'FACE_BOUND':'IDFB',
    'PRESENTATION_LAYER_ASSIGNMENT':'IDPLA',
    'SURFACE_STYLE_USAGE':'IDSSU',
    'SURFACE_SIDE_STYLE':'IDSSS',
    'SURFACE_STYLE_FILL_AREA': 'IDSSFA',
    'FILL_AREA_STYLE':'IDFAS',
    'FILL_AREA_STYLE_COLOUR':'IDFASC',
    'COLOUR_RGB': 'IDCRGB',
    'SHAPE_DEFINITION_REPRESENTATION': 'IDSDR',
    'PRODUCT_DEFINITION_SHAPE':'IDPDS',
    'PRODUCT_DEFINITION':'IDPD',
    'PRODUCT_DEFINITION_FORMATION_WITH_SPECIFIED_SOURCE':'IDPDFWSS',
    'PRODUCT':'IDPRD',
    'PRODUCT_CONTEXT':'IDPC',
    'APPLICATION_CONTEXT':'IDAC',
    'PRODUCT_DEFINITION_CONTEXT': 'IDPDC',
    'APPLICATION_PROTOCOL_DEFINITION':'IDAPD',
    'STYLED_ITEM': 'IDSI',
    'PRESENTATION_STYLE_ASSIGNMENT': 'IDPSA',
}


# Definiowanie zależności między encjami
dependency_order = [
    'CARTESIAN_POINT',
    'DIRECTION',
    'VECTOR',
    'LINE',
    'AXIS2_PLACEMENT_3D',
    'CIRCLE',
    'VERTEX_POINT',
    'B_SPLINE_CURVE_WITH_KNOTS',
    'EDGE_CURVE',
    'ORIENTED_EDGE',
    'PLANE',
    'CONICAL_SURFACE',
    'CYLINDRICAL_SURFACE',
    'SPHERICAL_SURFACE',
    'TOROIDAL_SURFACE',
    'ADVANCED_FACE',
    'CLOSED_SHELL',
    'MANIFOLD_SOLID_BREP',
    'EDGE_LOOP',
    'FACE_BOUND',
    'FACE_OUTER_BOUND',
    'APPLICATION_CONTEXT',
    'PRODUCT_CONTEXT',
    'APPLICATION_PROTOCOL_DEFINITION',
    'PRODUCT_DEFINITION_CONTEXT',
    'PRODUCT',
    'PRODUCT_DEFINITION_FORMATION_WITH_SPECIFIED_SOURCE',
    'PRODUCT_DEFINITION',
    'PRODUCT_DEFINITION_SHAPE',
    'ADVANCED_BREP_SHAPE_REPRESENTATION',
    'SHAPE_REPRESENTATION',
    'SHAPE_DEFINITION_REPRESENTATION',
    'COLOUR_RGB',
    'FILL_AREA_STYLE_COLOUR',
    'FILL_AREA_STYLE',
    'SURFACE_STYLE_FILL_AREA',
    'SURFACE_SIDE_STYLE',
    'SURFACE_STYLE_USAGE',
    'PRESENTATION_STYLE_ASSIGNMENT',
    'STYLED_ITEM',
    'PRESENTATION_LAYER_ASSIGNMENT',
    
]


# Globalny licznik dla identyfikatorów Part
current_part_id = 0

def generate_part_id():
    global current_part_id
    current_part_id += 1
    generated_id = f"Part{current_part_id:04d}"  # Generuje ID w formacie PartXXXX
    print(f"Wygenerowany identyfikator: {generated_id}")  # Wydrukowanie wygenerowanego identyfikatora
    return generated_id

part_id = generate_part_id()  # Generuje unikatowe IDPart dla tego pliku

# Funkcja konwertująca wartości '.T.' i '.F.' na wartości bitowe
def konwertuj_na_bit(wartosc):
    if wartosc in ('.T.', 'T'):
        return 1  # SQL Server rozpoznaje 1 jako wartość TRUE
    elif wartosc in ('.F.', 'F', '.U.'):
        return 0  # SQL Server rozpoznaje 0 jako wartość FALSE
    return wartosc  # Dla innych wartości, które nie wymagają konwersji

def przetwarzanie_linii(line, patterns, patternsAP214, part_id):
    for pattern_dict in [patterns, patternsAP214]:
        for entity, (pattern, columns) in pattern_dict.items():
            match = pattern.match(line)
            if match:
                data = list(match.groups())  # Wyciąganie danych z dopasowania.
                ein_with_hash = f"#{data[0]}"
                combined_patterns = {**patterns, **patternsAP214}
                # Konwersja pustych nazw na wartość domyślną
                data = [default_values.get(col, value) if value in ['$', '', ' '] else konwertuj_na_bit(value) for col, value in zip(combined_patterns[entity][1], data)]

                column_name = identifiers.get(entity, entity)[2:]  # Usuwamy 'ID' z nazwy kolumny
                entity_id = f"{part_id}_{column_name}_{ein_with_hash}"  # Nowy format ID
                print(f"Wygenerowany id: {entity_id}")
                
                # Przygotowujemy listę danych do DataFrame
                data_with_ids = [entity_id] + [ein_with_hash] + data[1:]  # Dla innych encji bez IDPart
               
                return entity, data_with_ids

    return None, None

def przetwarzaj_wartosci_listy(wartosc, ein_to_id_map):
    # Sprawdzamy, czy wartość jest typu string
    if isinstance(wartosc, str):
        if ',' in wartosc:
            # Usuwamy białe znaki z każdego elementu po podziale i mapujemy na ID
            return ','.join([ein_to_id_map.get(e.strip(), e.strip()) for e in wartosc.split(',')])
        return ein_to_id_map.get(wartosc, wartosc)
    else:
        # Jeśli wartość nie jest stringiem, zwracamy ją bez zmian
        return wartosc

def przetwarzaj_linie_i_generuj_df(lines, patterns, patternsAP214):
    ein_to_id_map = {}
    combined_patterns = {**patterns, **patternsAP214}
    dataframes = {
        entity: pd.DataFrame(columns=[identifiers[entity]] + ['EIN'] + columns[1:])
        for entity, (_, columns) in combined_patterns.items() if entity in identifiers
    }

    for line in lines:
        line = line.strip()
        entity, data = przetwarzanie_linii(line, patterns, patternsAP214, part_id)
        if entity:
            ein = data[1]  # EIN is always second in the list after entity_id
            ein_to_id_map[ein] = data[0]  # Map EIN to entity_id
            # Przygotowanie danych z domyślnymi wartościami
            try:
                dataframes[entity].loc[len(dataframes[entity])] = data
            except Exception as e:
                print(f"Error adding data to DataFrame: {e}")
                print(f"Data: {data}")

    # Aktualizacja EIN na ID w DataFrame'ach
    for df in dataframes.values():
        for col in df.columns:
            if col not in ['EIN', 'IDPart']:
                # Obsługa kolumn zawierających listy EIN
                df[col] = df[col].apply(lambda x: przetwarzaj_wartosci_listy(x, ein_to_id_map))

    return {entity: dataframes[entity] for entity in dependency_order}



# Funkcja do zapisywania danych do bazy danych
def zapisz_do_bazy(dataframes, engine):
    for entity, df in dataframes.items():
        for index, row in df.iterrows():
            try:
                # Tworzenie zapytania SQL sprawdzającego istnienie rekordu
                query = f"SELECT * FROM [{entity}] WHERE " + ' AND '.join([f"[{col}] = '{row[col]}'" for col in df.columns if col in row])

                # Sprawdź, czy rekord już istnieje w bazie danych przed wstawieniem
                if not pd.read_sql_query(query, con=engine).empty:
                    print(f"Rekord z {entity} o podanych wartościach już istnieje w bazie danych, pomijam.")
                    continue

                # Jeśli nie istnieje, wstaw rekord
                df.iloc[[index]].to_sql(entity, con=engine, if_exists='append', index=False)
                print(f"Dane dla {entity} zapisane pomyślnie do bazy danych.")
            except Exception as e:
                print(f"Błąd przy zapisywaniu danych dla {entity}: {e}")

# Funkcja do wczytywania i przetwarzania danych z pliku STEP
def wczytaj_i_przetwarzaj_dane(nazwa_pliku):
    with open(nazwa_pliku, 'r') as file:
        lines = file.readlines()

    przetworzone_linie = []
    temp_line = ''
    is_aggregating = False

    for line in lines:
        line = line.strip()
        if line.startswith('#'):
            if 'B_SPLINE_CURVE_WITH_KNOTS' in line and not is_aggregating:
                # Rozpoczęcie agregacji dla B_SPLINE_CURVE_WITH_KNOTS
                is_aggregating = True
                temp_line = line
            elif is_aggregating and not line.startswith('#'):
                # Kontynuacja agregacji linii dla B_SPLINE_CURVE_WITH_KNOTS
                temp_line += ' ' + line
            elif is_aggregating and line.startswith('#'):
                # Zakończenie agregacji i reset
                przetworzone_linie.append(temp_line)
                is_aggregating = False
                przetworzone_linie.append(line) # Dodanie aktualnej linii, która nie jest częścią B_SPLINE_CURVE_WITH_KNOTS
            else:
                przetworzone_linie.append(line)  # Dodanie standardowej linii
        elif is_aggregating:
            # Kontynuacja agregacji linii dla B_SPLINE_CURVE_WITH_KNOTS
            temp_line += ' ' + line

    # Dodanie ostatniej kumulowanej linii do listy, jeśli istnieje
    if temp_line:
        przetworzone_linie.append(temp_line)

    return przetworzone_linie

# Użycie funkcji
przetworzone_linie = wczytaj_i_przetwarzaj_dane(nazwa_pliku)


# Główna funkcja przetwarzania i zapisu danych
def przetwarzaj_i_zapisz_dane(nazwa_pliku, patterns,patternsAP214):
    przetworzone_linie = wczytaj_i_przetwarzaj_dane(nazwa_pliku)
    dataframes_in_order = przetwarzaj_linie_i_generuj_df(przetworzone_linie, patterns, patternsAP214)


    # Definiujemy mapowanie dla konkretnych encji i ich kolumn, które wymagają konwersji
    kolumny_do_konwersji = {
            'ADVANCED_FACE': ['same_sense'],
            'B_SPLINE_CURVE_WITH_KNOTS': ['closed_curve', 'self_intersect'],
            'EDGE_CURVE': ['same_sense'],

    }
        
    # Iterujemy przez każdy DataFrame i każdą kolumnę, która wymaga konwersji
    for entity, df in dataframes_in_order.items():
        if entity in kolumny_do_konwersji:
            for col in kolumny_do_konwersji[entity]:
                if col in df.columns:  # Upewniamy się, że kolumna istnieje
                        df[col] = df[col].apply(konwertuj_na_bit)  # Stosujemy konwersję
                        # Aktualizacja DataFrame w słowniku
                        dataframes_in_order[entity] = df
    
    
    # Zapisywanie do bazy danych w kolejności zależności
    zapisz_do_bazy(dataframes_in_order, engine)
    
    return dataframes_in_order



dataframes = przetwarzaj_i_zapisz_dane(nazwa_pliku, patterns,patternsAP214)



